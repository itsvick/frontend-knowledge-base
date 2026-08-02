# Code-Implementation

> Subsection of 30-Interview-Coding/JavaScript.

## Overview

These questions ask you to implement common utility functions from scratch — the interviewer wants to see you reason about closures, timers, and edge cases live, not just recall what a utility does.

## Key Concepts

- Debouncing and throttling both exist to stop a rapidly-firing event (typing, scrolling, resizing) from running an expensive handler too often — they differ in *how* they limit it.
- A closure-held variable (a timer ID, a "last run" timestamp) is what lets the wrapper function remember state across repeated calls.
- Preserving `this` and the call arguments (via `apply`/`call` or a rest parameter) is what makes the wrapper a transparent drop-in replacement for the original function.

## Interview Questions & Answers

### Q1. What are debouncing and throttling? Write a debounce function.

#### Answer

Both are techniques for controlling how often a function runs in response to a rapidly-firing event, but they limit it differently:

- **Debouncing** delays execution until the event stops firing for a specified period — only the *last* call in a burst actually runs. E.g. a search-as-you-type input: instead of firing an API call on every keystroke of "tree" (`T`, `Tr`, `Tre`, `Tree` — 4 calls), a 300ms debounce waits until the user stops typing and fires just once.
- **Throttling** guarantees a function runs at most once per specified interval, regardless of how many times the event fires — useful when the app needs to respond *periodically* during continuous activity rather than only at the end. E.g. a scroll handler throttled to 200ms runs on a steady cadence while the user keeps scrolling, instead of not at all until they stop.

In short: debounce waits for inactivity; throttle limits frequency during activity. That's also why debouncing suits search inputs (only the final value matters) while throttling suits scroll/resize handlers (the app needs updates *while* the activity is ongoing, not just after).

To implement debounce:
- Keep a `timeoutId` in the closure so it's remembered across calls to the returned wrapper.
- On every call, clear any pending timer with `clearTimeout` — this is what cancels the previous scheduled run whenever the event fires again before the delay elapses.
- Schedule a new `setTimeout` that calls the original function after `delay` ms.
- Use `apply()` (or a rest parameter with an arrow function) so the original function still receives the correct `this` and arguments.

This is the **trailing-edge** debounce (runs after the burst ends) — the common default. A **leading-edge** debounce instead runs immediately on the first call, then ignores subsequent calls until the delay passes; libraries like Lodash's `_.debounce()`/`_.throttle()` expose both via `leading`/`trailing`/`maxWait` options.

#### Code Example

```js
function debounce(fn, delay) {
  let timeoutId;
  return function (...args) {
    const context = this;
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  };
}

// Usage
const debouncedSearch = debounce((query) => fetchResults(query), 300);
searchInput.addEventListener("input", (e) => debouncedSearch(e.target.value));
```

```js
// Throttle, for comparison — runs at most once per `interval`
function throttle(fn, interval) {
  let lastRun = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastRun >= interval) {
      lastRun = now;
      fn.apply(this, args);
    }
  };
}
```

#### Follow-up Questions

- Why is debouncing generally preferred for search inputs, but throttling preferred for scroll/resize handlers?
- How would you add a `leading` option so the debounced function can fire immediately on the first call instead of waiting?
- How does `maxWait` (as in Lodash's `_.debounce`) change trailing-edge debounce behavior when calls never stop coming?

## Common Pitfalls

- Losing `this`/arguments in the wrapper — forgetting `apply()`/`call()` (or a rest parameter) means the debounced/throttled function silently drops arguments or runs with the wrong context.
- Creating a new debounced function on every render/call (e.g. `onChange={debounce(fn, 300)}` inline in a component) instead of once outside — this resets the closure's timer state every time, so debouncing never actually happens.
- Using debounce where throttle is needed (or vice versa) — e.g. debouncing a scroll handler means it may never fire at all during continuous scrolling.
