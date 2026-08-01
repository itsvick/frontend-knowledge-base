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

**Answer:**
- Synchronous: Executes line by line.
- Asynchronous: Doesn't block execution (e.g., `setTimeout`, Promises, `async`/`await`).

### Q2. What are async and await?

**Answer:** They simplify working with Promises.

## Code Examples

```js
// Q1: sync vs async
console.log("1");
setTimeout(() => console.log("2"), 0); // deferred to the event loop
console.log("3");
// Output: 1, 3, 2

// Q2: async/await
async function getData() {
  const res = await fetch(url);
}
```

## Common Pitfalls

- Assuming `setTimeout(fn, 0)` runs immediately — it still waits for the current call stack to clear.
- Forgetting `try`/`catch` around `await`, so rejected Promises become unhandled errors.
- Awaiting Promises sequentially when they could run concurrently with `Promise.all`.

## References

-
