# Closures

> Subsection of 01-JavaScript.

## Overview

A closure is a function that remembers variables from its outer (lexical) scope even after the outer function has finished executing.

## Key Concepts

- Formed whenever an inner function is defined inside an outer function and references the outer function's variables
- The closed-over variables persist in memory as long as the inner function is reachable
- Commonly used for data privacy, counters, and memoization

## Interview Questions & Answers

### Q1. What is a closure?

**Answer:** A closure is a function that remembers variables from its outer scope even after the outer function has finished executing.

## Code Examples

```js
function outer() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}

const counter = outer();
counter(); // 1
counter(); // 2
```

## Common Pitfalls

- Creating closures over a shared `var` inside loops, so every callback captures the same final value instead of a per-iteration one (fixed by using `let`).
- Unintentionally keeping large objects alive in memory because a closure still references them.

## References

-
