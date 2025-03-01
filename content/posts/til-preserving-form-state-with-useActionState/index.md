---
title: "TIL: Preserving Form State with React's useActionState Hook"
date: 2025-03-01
draft: false
---

React recently introduced the [`useActionState`](https://react.dev/reference/react/useActionState) hook, and while the official docs provide basic examples, I immediately wanted to tackle a common real-world scenario: preserving form input values after submission errors.

## The Problem

When a user submits a form with invalid credentials, we typically want to:

1. Show an error message
2. Preserve what they've already typed (especially for multi-field forms)

Without this, users face the frustration of re-typing information they've already entered.

## Traditional Approach

The traditional approach requires maintaining controlled components with `onChange` handlers:

```jsx
function SignInForm() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [error, setError] = useState(null)

  const handleSubmit = async (e) => {
    e.preventDefault()
    try {
      await signIn(email, password)
    } catch (err) {
      setError('Invalid credentials')
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && <div className="error">{error}</div>}
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Sign In</button>
    </form>
  )
}
```

## The `useActionState` Approach

With `useActionState`, we can achieve the same result with less code and without tracking intermediate states:

```js
'use server'

async function signInAction(prevState, formData) {
  const email = formData.get('email')
  const password = formData.get('password')

  try {
    await signIn(email, password)
    return { success: true }
  } catch (error) {
    return { 
      email, 
      error: 'Invalid credentials' 
    }
  }
}
```

```jsx
import { useActionState } from 'react'

function SignInForm() {
  const [state, formAction] = useActionState(signInAction, {
    // initial state
  })

  if (state?.success) {
    return <div>You're signed in!</div>
  }

  return (
    <form action={formAction}>
      {state?.error && <div className="error">{state.error}</div>}
      
      <input
        type="email"
        name="email"
        defaultValue={state?.email || ''}
        required
      />

      <input
        type="password"
        name="password"
        required
      />

      <button type="submit">Sign In</button>
    </form>
  )
}
```

The key part is to set the _defaultValue_ on the _input_.

In particular, I'm a fan on letting the native components do its job. Historically, React (or people using React) used to track every piece of state - think of the state of _input_, kept in _state_ and tracked with _onChange_. I'm happy with the paradigm shift.
