# Promises

> Subsection of 01-JavaScript.

## Overview

A Promise represents the eventual completion or failure of an asynchronous operation, exposing `.then()`/`.catch()`/`.finally()` to handle the result.

## Key Concepts

- Three states: pending, fulfilled, rejected
- Once settled, a Promise's state is immutable
- `.then()` chains transformations; `.catch()` handles rejections

## Interview Questions & Answers

### Q1. What is a Promise?

#### Answer

A Promise represents the eventual completion or failure of an asynchronous operation. Before Promises, asynchronous code relied on callbacks, but chaining multiple callbacks together (e.g. one async step depending on the previous one) quickly became unmanageable — often called "callback hell."

A Promise has three states:
- **Pending** — the initial state; the async operation hasn't completed yet.
- **Fulfilled** — the operation completed successfully.
- **Rejected** — the operation failed.

Once a Promise moves to fulfilled or rejected it's called **settled** — "settled" isn't a separate state, just an umbrella term meaning "no longer pending," and a settled Promise's outcome is final and can't change again.

A Promise is created via the `Promise` constructor, which takes an executor callback with two functions as parameters: `resolve` (called when the operation succeeds) and `reject` (called when it fails or errors). The result is consumed with `.then()` (runs on fulfillment) and `.catch()` (runs on rejection).

#### Code Example

```js
function sumOfThreeElements(...elements) {
  return new Promise((resolve, reject) => {
    if (elements.length > 3) {
      reject("Only three elements or less are allowed");
    } else {
      let sum = 0;
      for (let i = 0; i < elements.length; i++) sum += elements[i];
      resolve("Sum has been calculated: " + sum);
    }
  });
}

sumOfThreeElements(4, 5, 6)
  .then((result) => console.log(result)) // fulfilled — "Sum has been calculated: 15"
  .catch((error) => console.log(error));

sumOfThreeElements(7, 0, 33, 41)
  .then((result) => console.log(result))
  .catch((error) => console.log(error)); // rejected — "Only three elements or less are allowed"

// A real-world example — fetch() returns a Promise
fetch(url)
  .then((res) => res.json())
  .catch((err) => console.log(err));
```

#### Follow-up Questions

- How do `Promise.all()`, `Promise.race()`, and `Promise.allSettled()` differ?
- Why is a Promise's state immutable once settled?
- How does `async`/`await` relate to Promises under the hood?

## Common Pitfalls

- Forgetting to return a value/Promise inside a `.then()` chain, breaking the chain.
- Not adding a `.catch()`, leading to unhandled Promise rejections.
