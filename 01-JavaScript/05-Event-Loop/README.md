# Event-Loop

> Subsection of 01-JavaScript.

## Overview

How JavaScript — a single-threaded language — handles asynchronous work without blocking: the call stack runs synchronous code, Web APIs handle async operations in the background, and the event loop decides what runs next once the stack is clear.

## Key Concepts

- **Call stack** — where synchronous code actually executes, one frame at a time
- **Web APIs** — the browser environment (not the JS engine itself) that manages timers, network requests, and DOM event listeners in the background
- **Macrotask queue** — holds timer callbacks, DOM events, and I/O callbacks
- **Microtask queue** — holds Promise callbacks and `queueMicrotask()` callbacks
- The event loop's core rule: once the call stack is empty, it drains the **entire microtask queue** before running even one macrotask

## Interview Questions & Answers

### Q1. How does the JavaScript event loop work?

#### Answer

JavaScript is single-threaded — only one thing runs on the call stack at a time — so the event loop is what lets it handle asynchronous work (timers, network requests, DOM events) without blocking. Four pieces are involved: the **call stack**, the browser's **Web APIs** environment, the **macrotask queue**, and the **microtask queue**.

Walking through an example: a script logs `"1"`, schedules a `setTimeout` to log `"2"`, schedules a resolved Promise to log `"3"`, then synchronously logs `"4"`.

1. Synchronous code runs directly on the call stack, in order, so `"1"` logs immediately.
2. `setTimeout` isn't run on the call stack itself — its timer is handed off to the Web APIs environment, which manages timers, network requests, and DOM event listeners outside the JS engine.
3. `Promise.resolve().then(...)` schedules its callback on the **microtask queue**, not the general task queue.
4. The last synchronous line runs, logging `"4"`. The call stack is now empty — this is where the event loop takes over.

The event loop continuously checks whether the call stack is empty, and if so, decides what queued work runs next. Its critical rule: **the microtask queue is always fully drained before the next macrotask runs**. So once the stack is empty, the Promise callback runs first, logging `"3"`. Only after the microtask queue is completely empty does the event loop move to the macrotask queue, running the timer callback and logging `"2"`.

Final output: `1, 4, 3, 2`.

This is also why `setTimeout(fn, 0)` never runs immediately, even with a zero delay — the callback still has to wait for the current call stack to finish and for all pending microtasks to drain first.

#### Code Example

```js
console.log("1");

setTimeout(() => console.log("2"), 0); // macrotask — deferred to Web APIs, then the macrotask queue

Promise.resolve().then(() => console.log("3")); // microtask — runs before any macrotask

console.log("4");

// Output: 1, 4, 3, 2
```

#### Follow-up Questions

- What happens if a microtask schedules another microtask?

  The event loop keeps draining the microtask queue until it's completely empty before moving on to the next macrotask — including any new microtasks scheduled *while* draining. In extreme cases (e.g. a microtask that keeps scheduling more microtasks), this can starve macrotasks indefinitely and even block rendering in the browser.

- Where does rendering fit into this cycle relative to microtasks and macrotasks?
- How do `async`/`await` map onto the microtask queue under the hood?

## Common Pitfalls

- Assuming `setTimeout(fn, 0)` runs immediately — it still waits for the call stack to clear and all pending microtasks to drain first.
- Assuming Promise callbacks and `setTimeout` callbacks run in the order they were scheduled — microtasks (Promises) always run before the next macrotask (timers), regardless of a `0` delay.
- Scheduling microtasks recursively without a way to stop, which can starve macrotasks and freeze rendering.
