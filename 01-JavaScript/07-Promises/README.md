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

A Promise represents the eventual completion or failure of an asynchronous operation.

#### Code Example

```js
fetch(url)
  .then((res) => res.json())
  .catch((err) => console.log(err));
```

## Common Pitfalls

- Forgetting to return a value/Promise inside a `.then()` chain, breaking the chain.
- Not adding a `.catch()`, leading to unhandled Promise rejections.
