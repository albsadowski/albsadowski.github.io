---
title: "Typed Configuration Object in TypeScript"
date: 2023-07-16
draft: false
---

## Compile-Time vs Runtime in TypeScript

_TypeScript_ is an amazing technology, but it has been build under the assumption is compiles down to _JavaScript_. As a consequence, all the wonderful types one can enjoy during the compile time, are completely gone during the runtime. Thus, it's crucial to validate all the data from the IO.

As the types doesn't leave us any information about the data at the runtime, we usually end up maintaining two definitions of the same: (1) the class/interface for the typing purposes (2) JSON schema for validation purposes.

There are many libraries/attempts to help with the problem, but my favourite is [Ajv](https://ajv.js.org/). Ajv can work as a standard JSON schema validation tool, but it has a superpower of making the [JSON Schema](https://ajv.js.org/json-schema.html)/[JSON Type Definition](https://ajv.js.org/json-type-definition.html) visible to TypeScript, i.e., from the JSON Schema/Type Def defined in the codebase, Ajv can transform that object into a TypeScript `type` at the compile-time, e.g.:

```ts
import { JTDDataType } from "ajv/dist/jtd"

const FooSchema = {
    properties: {
        foo: {
            type: "string"
        },
    }
} as const

type Foo = JTDDataType<typeof FooSchema>
```

The `FooSchema` definition can be used for the actual validation, and the `Foo` is a type definition we can use in our codebase, thus we don't need an additional `interface`/`class` re-definition.

## Using Ajv for Configuration Object

When writing a nodejs application, one of the first dynamically typed pieces of information to deal with is configuration. We either load from a file, environment or maybe a secrets manager, but it's all guaranteed to be untyped by default.

With Ajv, one can define a configuration schema, leverage that for validation purposes and use the magic of type transformation to get the typings for the schema.

As there's some boilerplate around reading files/environment, I ended up extracing that into a small, Ajv-based, library [@dumpstate/config](https://github.com/dumpstate/config).

From now on, when I deal with configuration in nodejs, my code looks as follows:

```ts
import { ConfigSchemaType, loadConfig } from "@dumpstate/config"

// NB I prefer JSON Type Definition
const ConfigSchema = {
    properties: {
        app: {
            properties: {
                host: { type: "string" },
                port: { type: "int32" },
            },
        },
    },
} as const

// NB ConfigSchemaType is just convenience proxy to JTDDataType from Ajv
type Config = ConfigSchemaType<typeof ConfigSchema>

// NB config is of `Config` type - both typed and validated
const config = loadConfig(ConfigSchema, { appName: "foo" })
```

`loadConfig` supports two configuration loaders (in the order of loading):
1. file configuration loader:
* loads `config/default.application.json`,
* loads `config/application.json`,
* loads any file provided as `APPLICATION_CONFIG` environment variable.
2. environment configuration loader:
* expects environment variables in the format e.g., `${appName}__APP__HOST`, where the `appName` is the name of the application passed to the `loadConfig`, then all `.` on the JSON path are transformed into `__` and camel cased properties are snake cased.

## Links

1. [Ajv](https://ajv.js.org/)
2. [@dumpstate/config](https://github.com/dumpstate/config)
