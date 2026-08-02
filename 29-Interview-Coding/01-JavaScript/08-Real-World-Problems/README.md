# Real-World-Problems

> Subsection of 30-Interview-Coding/JavaScript.

## Overview

These are practical, system-facing coding problems that show up in machine-coding rounds: building small pieces of infrastructure (rate limiters, retry logic, concurrency control) rather than pure algorithms.

## Key Concepts

- A queue-based design (track an "in-flight" count, pull the next item when one finishes) is the common pattern behind rate limiters, concurrency limiters, and job schedulers.
- Starting async work eagerly (e.g. mapping an array straight into invoked promises) defeats any limiter — the work already began before it was ever "queued."

## Interview Questions & Answers

### Q1. How would you design a concurrency limiter for async tasks?

#### Answer

A concurrency limiter controls how many promises are allowed to run at the same time. Instead of firing all async tasks at once (e.g. `Promise.all` over hundreds of `fetch` calls), only a fixed number (`limit`) execute concurrently, while the rest wait in a queue. When one task finishes, the next queued task is started immediately, so the number of in-flight tasks never exceeds `limit`.

This is done by:
- Keeping an index/pointer into the pending tasks and an active-count.
- Starting tasks up to `limit` immediately.
- On each task's completion (success or failure), decrementing the active count and pulling the next task off the queue.

This prevents overloading an API (hitting rate limits) or the browser (too many concurrent connections/DOM work), while still processing everything as fast as the limit allows — instead of running everything serially (slow) or all at once (unsafe).

#### Code Example

```js
function runWithConcurrency(taskFns, limit) {
  return new Promise((resolve, reject) => {
    const results = new Array(taskFns.length);
    let nextIndex = 0;
    let completed = 0;

    if (taskFns.length === 0) return resolve(results);

    function runNext() {
      if (nextIndex >= taskFns.length) return;
      const currentIndex = nextIndex++;

      taskFns[currentIndex]()
        .then((result) => {
          results[currentIndex] = result;
          completed++;
          if (completed === taskFns.length) {
            resolve(results);
          } else {
            runNext(); // pull the next queued task into this freed slot
          }
        })
        .catch(reject);
    }

    // Kick off only `limit` tasks initially — the rest stay queued
    const initialBatch = Math.min(limit, taskFns.length);
    for (let i = 0; i < initialBatch; i++) runNext();
  });
}

// Usage — each entry is a function that RETURNS a promise, not an already-started one
const taskFns = urls.map((url) => () => fetch(url).then((res) => res.json()));

runWithConcurrency(taskFns, 3).then((results) => console.log(results));
```

#### Follow-up Questions

- How would you make this a reusable wrapper (like `p-limit`) that limits any async call site, not just a fixed array of tasks?
- How would you use `Promise.allSettled` semantics here so one failing task doesn't reject the whole batch?
- How would you add a per-task timeout or the ability to cancel queued/in-flight tasks?

## Common Pitfalls

- Passing already-invoked promises (e.g. `urls.map(url => fetch(url))`) instead of task *functions* — the requests all fire immediately regardless of `limit`, since the async work starts the moment `fetch` is called, not when it's awaited.
- Forgetting to track `completed` count separately from `nextIndex` — resolving too early (before all tasks finish) or never resolving at all.
- Not preserving output order: results should map back to the original input order even though tasks may resolve out of sequence.
