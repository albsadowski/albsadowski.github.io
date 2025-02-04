---
title: "Reader Monad for DB Connection in TypeScript"
date: 2023-05-29
draft: false
---

## MVC(S)

The _Model-View-Controller_ is usually a standard pattern for building web applications. Whether the _view_ is expressed as HTML or perhaps just plain JSON, it helps to structure the application in an elegant way.

The _Model_ is a set of components responsible for representing the persistence layer, often in the form of some ORM classes. _View_ maintains the recipe for generating the _view_ given the _model_. Finally _controller_ glues both parts together and deals with the other problems inherent in web development, such as parsing and interpreting query parameters.

Where does the business logic live? As long as the application is trivial, the _model_ representation, combined with the power of ORMs, provides a powerful tool for manifesting the business domain. Then, the actual logic, either leaks to the _controller_ or requires another component in the MVC pattern.

The _service_ could be one of these components. The _service_ sits between the _controller_ and the _model_, either by contructing higher level _model_ objects (non-ORM), or by simply enriching the ORM objects with an additional computation.

## ACID

As the application grows, the _services_ become complex, often forming a hierarchy. Assuming there's still a database-backed _model_ under the hood, all the _services_ do is run a sequence of database queries to either read or mutate the state of the system.

```ts
async function controllerMethod() {
    const res1 = await this.service1()
    return this.service2(res1)
}
```

How can we ensure that a _controller_ calling one or more _services_, maintains the _transation_ boundary? We need to make all the components aware of the current state of the transaction. The current transaction state is usually just an instance of a _database connection_. And that's the moment where an implementation _detail_ of the _model_ brutally leaks through the MVC(S) stack.

If you want to make a _service_ transactional, it should accept a _database connection_ as a parameter on every method of a _service_. This is inconvenient and tightly couples all of the web application code to a chosen database (+ probably a chosen ORM/similar).

```ts
async function controllerMethod() {
    const tr = await dbPool.transact()

    try {
        const res1 = await this.service1(tr)
        const res2 = await this.service2(tr, res1)
        await tr.commit()
        return res2
    } catch {
        await tr.rollback()
    }
}
```

Another way to solve this problem via the state of the _request_ - but the problem is equivalent - now we either inject the _request_ or couple application code with a chosen web framework.

The approach suggested in this blog post is the _monad way_.

## The Monad Way

The [_Reader monad_](https://hackage.haskell.org/package/mtl-2.3.1/docs/Control-Monad-Reader.html) is a well known pattern in the _Haskell_ community to solve the problem of a shared _environment_ between several components of the system.

The idea is quite simple:
1. do not require the _environment_ until you actually need it,
2. use _monad_ properties to compose functions that depends on the _environment_.

In our case, the _environment_ is a database connection - either raw connection, or a transaction.

Point (1) above, is easier said than done, but in practice we can describe the whole computation process as a chain of lazily evaluated functions. Then, the final artefact should be a function that actually executes the program given the _environment_ (database connection).

For _TypeScript_ I have extracted the boilerplate into [_DBAction_](https://github.com/albsadowski/dbaction) library, an example:

```ts
async function controllerMethod() {
    return this.service1()
        .flatMap(res => this.service2(res))
        .transact(this.transactor)
}
```

Let's break this down:

### DBAction

_DBAction_ is an actual _reader monad_ for the database connection. What services return is not a `Promise<Result>` type (a _promise_ of some kind of result), but `DBAction<Result>`. After the service is called `this.service1()`, nothing happens until we either `run` or `transact` the `DBAction`. The value is referentially transparent, so if you call `this.service1` with the same input, you'll always get the same result.

The cost of this approach, is that all the components involved in the chain, must return _DBActions_. On the implementation side, this means that the moment you actually need the database connection, you should wrap it in a function that takes _database connection_ as its only argument and returns a _promise_ of some result type, e.g.:

```ts
const service1 = new DBAction((conn) => conn.query("select..."))
```

There are two methods available on the _DBAction_ that should help composing the monad:
1. `.map<K>(fn: (item: T) => K)` - mapping inner value into another value,
2. `.flatMap<K>(fn: (item: T) => DBAction<K>)` - mapping inner value into another value, where the `fn` function also require the _database connection_ for the computation.

The _DBAction_ can be:
1. `.run(tr: Transactor): Promise<T>` - meaning the transactor will inject plain database connection into the chain,
2. `.transact(tr: Transactor): Promise<T>` - the transactor will start the transaction and then inject the connection into the chain, finally _commit_ or _rollback_.

### Transactor

The _transactor_ is a component that knows:
1. the type of the database connection,
2. the details of establishing the database connection, or requesting the connection from the pool,
3. the details of executing the query and/or managing the transaction context.

The _transactor_ is database specific and depends on the database driver. There should be a single transactor for the entire application.

As of now, _DBAction_ offers the one for _PostgreSQL_:

```ts
import { Transactor } from "@dumpstate/dbaction/lib/PG"

// `pool` - an instance of `pg` connection pool
const tr = new Transactor(pool)
```

### Utilities

The utilities available in the library, useful for composing the monad:
1. `flatten` - tranforms an array of `DBAction`s into a `DBAction` of an array of values,
2. `pure` - wraps a scalar / promise / function returning a promise with a `DBAction` - useful to lift non-_DBAction_ into a _DBAction_.
3. `chain` - builds a sequential chain of _DBActions_ - result of a previous is an argument for the latter,
4. `sequence` - runs operations concurrently; returns a single `DBAction`,
5. `query` - creates a _DBAction_ for a query string.

## Links

1. [_DBAction_ GitHub Repository](https://github.com/albsadowski/dbaction).
2. [_Reader Monad_](https://hackage.haskell.org/package/mtl-2.3.1/docs/Control-Monad-Reader.html).
3. Heavily inspired by [_doobie_](https://tpolecat.github.io/doobie/).
