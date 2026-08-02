# Polyfills

> Subsection of 30-Interview-Coding/JavaScript.

## Overview

These questions ask you to reimplement a built-in method from scratch — the point isn't the trivia of what the method does, but showing you understand it well enough to reproduce its exact behavior (parameters, return value, edge cases) with plain loops.

## Key Concepts

- `map()`, `filter()`, and `reduce()` all iterate an array without mutating it, but differ in what they return: `map()` transforms each element into a new array of the same length, `filter()` selects a subset of elements into a (possibly shorter) new array, `reduce()` collapses the whole array into a single accumulated value.
- Implementing a polyfill is mostly: loop through the array, call the callback with the right arguments (`element, index, array`), and build up the result the same way the native method would.

## Interview Questions & Answers

### Q1. Explain map(), filter(), and reduce() with examples. Implement your own map().

#### Answer

- **`map()`** applies a function to every element and returns a **new array** of the transformed values, same length as the original — the source array is left untouched.
- **`filter()`** returns a new array containing only the elements for which the callback returns a truthy value — it selects elements rather than transforming them, so the result can be shorter than the original.
- **`reduce()`** collapses the entire array into a **single value** by repeatedly applying a callback that carries an accumulator (`acc`) forward across iterations, starting from an initial value.

A common mistake with `reduce()` is omitting the initial value — without it, the first array element becomes the initial accumulator, which usually still works but throws on an empty array (since there'd be no element to seed `acc` with).

These three compose well in a chain — e.g. filtering active users, mapping to their names, then reducing to a display string.

To implement `map()` yourself: loop over the array, call the callback with `(element, index, array)` for each one, and push the result into a new array.

`map()` vs `forEach()` — see [[01-JavaScript/01-Basics]] Q12: `map()` returns a new array, `forEach()` returns `undefined`, so reaching for `map()` only makes sense when you intend to use its returned array.

#### Code Example

```js
const numbers = [1, 2, 3, 4];

const doubled = numbers.map((num) => num * 2);           // [2, 4, 6, 8] — transforms every element
const evenNumbers = numbers.filter((num) => num % 2 === 0); // [2, 4] — selects matching elements
const sum = numbers.reduce((acc, num) => acc + num, 0);     // 10 — collapses to one value

// Chaining — filter, then map, then reduce
const result = users
  .filter((user) => user.isActive)
  .map((user) => user.name)
  .reduce((acc, name) => acc + ", " + name, "");

// Implementing your own map()
function myMap(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(callback(arr[i], i, arr));
  }
  return result;
}

myMap([1, 2, 3], (num) => num * 2); // [2, 4, 6]
```

#### Follow-up Questions

- Why does omitting `reduce()`'s initial value break on an empty array?
- How would you implement `filter()` or `reduce()` yourself, following the same pattern as `myMap`?
- How would your `myMap` handle a sparse array (e.g. `[1, , 3]`) compared to the native `Array.prototype.map`?

## Common Pitfalls

- Omitting `reduce()`'s initial value, which silently breaks on an empty array and can produce the wrong result when the array has only one element.
- Using `map()` when the return value is discarded — `forEach()` communicates intent better when you're only iterating for side effects.
- Forgetting that `map()`/`filter()`/`reduce()` don't mutate the original array — reassigning the result is required, not just calling the method.
