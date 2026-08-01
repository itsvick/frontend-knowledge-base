# Interview-Coding

> Subsection of 01-JavaScript.

## Overview

Practical array/object method questions commonly asked in coding interviews.

## Key Concepts

- `map()` transforms every element and returns a new array of the same length
- `forEach()` runs a side effect per element and returns `undefined`
- `filter()` returns a new array with all elements matching a condition
- `find()` returns only the first element matching a condition (or `undefined`)

## Interview Questions & Answers

### Q1. What is the difference between map() and forEach()?

**Answer:**
- `map()` returns a new array.
- `forEach()` does not return a new array.

### Q2. What is the difference between filter() and find()?

**Answer:**
- `filter()` returns all matching elements.
- `find()` returns the first matching element.

## Code Examples

```js
const nums = [1, 2, 3, 4];

const doubled = nums.map((n) => n * 2);      // [2, 4, 6, 8]
nums.forEach((n) => console.log(n));         // logs each, returns undefined

const evens = nums.filter((n) => n % 2 === 0); // [2, 4]
const firstEven = nums.find((n) => n % 2 === 0); // 2
```

## Common Pitfalls

- Using `map()` purely for side effects (like `forEach`) and discarding its return value — wastes the new array allocation.
- Assuming `find()` returns an array like `filter()` — it returns a single element or `undefined`.

## References

-
