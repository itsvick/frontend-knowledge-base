# Async

> Subsection of 01-JavaScript.

## Overview

How JavaScript handles operations that don't complete immediately: the sync/async distinction and the `async`/`await` syntax built on Promises.

## Key Concepts

- Synchronous code executes line by line, blocking further execution
- Asynchronous code doesn't block execution — results arrive later via the event loop
- `async` functions always return a Promise; `await` pauses execution until it settles
- Errors from a rejected awaited Promise are caught with `try`/`catch`

## Interview Questions & Answers

### Q1. What is the difference between synchronous and asynchronous JavaScript?

#### Answer

- Synchronous: Executes line by line.
- Asynchronous: Doesn't block execution (e.g., `setTimeout`, Promises, `async`/`await`).

#### Code Example

```js
console.log("1");
setTimeout(() => console.log("2"), 0); // deferred to the event loop
console.log("3");
// Output: 1, 3, 2
```

### Q2. How do async/await work, and how do you handle errors with them?

#### Answer

`async`/`await` is syntax built on top of Promises (see [[01-JavaScript/07-Promises]]) that lets asynchronous code read like synchronous code, avoiding `.then()`/`.catch()` chaining:

- **`async`** — placing it before a function makes that function always return a Promise. Returning a plain value (e.g. `return 10;`) is automatically wrapped as `Promise { 10 }`.
- **`await`** — used inside an `async` function to pause that function's execution until the awaited Promise settles, then unwraps the resolved value directly (instead of needing a `.then()` callback). It only pauses the `async` function itself, not the rest of the program — the surrounding code keeps running via the event loop.

**Error handling** is done with `try`/`catch`, which behaves the same way it would for synchronous throws — a rejected awaited Promise is caught in the `catch` block, equivalent to a Promise's `.catch()`:

```js
async function getUsers() {
  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/users");
    const data = await response.json();
    return data;
  } catch (error) {
    console.log("Something went wrong:", error);
  }
}
```

**Sequential vs parallel execution** is a common follow-up: awaiting one call at a time inside a `for...of` loop runs requests sequentially (each waits for the previous one, slower but safe when calls depend on each other), while mapping to an array of promises and awaiting `Promise.all` runs them in parallel (faster, but only safe when the calls are independent).

```js
// Sequential — each request waits for the previous one
for (const user of users) {
  await fetch(user.url);
}

// Parallel — all requests run together
const promises = users.map((user) => fetch(user.url));
await Promise.all(promises);
```

One caveat with `Promise.all`: if even one Promise in the batch rejects, the whole `Promise.all` rejects immediately, discarding the other results. When partial results are acceptable, `Promise.allSettled()` is used instead — it resolves with each Promise's individual status (fulfilled/rejected) rather than failing the whole batch.

#### Code Example

```js
async function getData() {
  const res = await fetch(url);
  const data = await res.json();
  return data;
}
```

#### Follow-up Questions

- Why does `await` inside `.forEach()` not work as expected, and why should `for...of` be used instead for sequential async work?
- When would you choose `Promise.allSettled()` over `Promise.all()`?

## Common Pitfalls

- Assuming `setTimeout(fn, 0)` runs immediately — it still waits for the current call stack to clear.
- Forgetting `try`/`catch` around `await`, so rejected Promises become unhandled errors.
- Awaiting Promises sequentially when they could run concurrently with `Promise.all`.
- Forgetting `await` entirely, leaving a dangling pending Promise instead of its resolved value — especially easy to miss inside conditionals.
- Using `await` inside `.forEach()`, expecting sequential execution — `forEach` doesn't wait for the callback's returned Promise, so all iterations fire without waiting; `for...of` should be used instead.
- Reaching for `Promise.all` without considering that one rejection fails the entire batch — `Promise.allSettled()` should be used when partial results are acceptable.
