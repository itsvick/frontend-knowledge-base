# Generators

> Subsection of 01-JavaScript.

## Overview

Generator functions can pause execution partway through and resume later, making them useful for building custom iterators and lazily-produced sequences.

## Key Concepts

- Declared with `function*`, and don't run their body immediately when called — calling one returns a generator object instead
- `yield` pauses execution and produces a value; execution resumes from that point on the next `next()` call
- `next()` returns `{ value, done }` — `value` is the yielded (or returned) value, `done` is `true` once the function has fully finished
- A generator's `return` statement supplies the final `value` and sets `done: true`

## Interview Questions & Answers

### Q1. What are generator functions?

#### Answer

Generator functions, introduced in ES6, are a special kind of function that can be paused midway through execution and resumed later, picking up exactly where they left off. They're declared with `function*` instead of `function`.

Unlike a normal function — which runs its body immediately when called and stops as soon as it hits `return` — calling a generator function doesn't execute its body at all. Instead it returns a **generator object**, which controls the execution via its `next()` method. Each call to `next()` runs the function up to the nearest `yield` statement, pausing there and returning `{ value, done: false }`, where `value` is whatever was yielded. Once the function runs to completion (hits `return` or falls off the end), `next()` returns `{ value, done: true }`, where `value` is the returned value (or `undefined` if none).

This pause/resume behavior makes generators well-suited for building custom iterators — sequences of values produced one at a time, on demand, rather than all at once.

#### Code Example

```js
// A normal function stops entirely at "return"
function normalFunc() {
  return 22;
  console.log(2); // never runs
}

// A generator function doesn't run its body until next() is called
function* genFunc() {
  yield 3;
  yield 4;
}
genFunc(); // returns a Generator object, doesn't execute anything yet

genFunc().next(); // { value: 3, done: false } — paused at the first yield

// Using a generator to build a custom iterator
function* iteratorFunc() {
  let count = 0;
  for (let i = 0; i < 2; i++) {
    count++;
    yield i;
  }
  return count;
}

let iterator = iteratorFunc();
console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: true } — hit the return statement
```

#### Follow-up Questions

- How do generators relate to the iterable protocol (`Symbol.iterator`)?
- How can generators be used to implement lazy evaluation of infinite sequences?
- How does `yield` differ from `yield*`?

## Common Pitfalls

- Assuming calling a generator function runs its body immediately, like a normal function — it only returns a generator object until `next()` is called.
- Forgetting that a generator's `return` statement still shows up as a final `{ value, done: true }` from `next()`, not as a thrown error or a no-op.
