# Advanced

> Subsection of 01-JavaScript.

## Overview

Advanced JavaScript techniques that build on closures and first-class functions, such as transforming how functions are invoked.

## Key Concepts

- Currying transforms a function of `n` arguments into `n` functions that each take one (or fewer) arguments
- Currying doesn't change what a function does — only how it's invoked
- A generic curry helper can be written once and reused for any function, regardless of its argument count

## Interview Questions & Answers

### Q1. What is currying in JavaScript?

#### Answer

Currying is a technique that transforms a function of `n` arguments into a sequence of `n` functions, each taking a single (or fewer) argument. A function `f(a, b)` becomes `f(a)(b)` — currying doesn't change the function's behavior, only how it's invoked, one argument at a time.

#### Code Example

```js
function add(a) {
  return function (b) {
    return a + b;
  };
}
add(3)(4); // 7

// Turning an existing two-argument function into its curried form
function multiply(a, b) {
  return a * b;
}
function currying(fn) {
  return function (a) {
    return function (b) {
      return fn(a, b);
    };
  };
}
var curriedMultiply = currying(multiply);
multiply(4, 3);        // 12
curriedMultiply(4)(3); // 12 — same result, invoked one argument at a time
```

A generic curry helper can be written once and reused for a function with any number of arguments, by comparing how many arguments have been collected so far against the target function's declared arity (`fn.length`):

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function (...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

function sum(a, b, c) {
  return a + b + c;
}
var curriedSum = curry(sum);
curriedSum(1)(2)(3);   // 6
curriedSum(1, 2)(3);   // 6 — arguments can be supplied in any grouping
curriedSum(1, 2, 3);   // 6 — or all at once
```

#### Follow-up Questions

- What's a real-world use case for currying (e.g. partial application, building specialized functions from a generic one)?
- How does currying differ from partial application?
- Why does the generic `curry()` helper check `fn.length` instead of a fixed argument count?

## Common Pitfalls

- Writing a generic `curry()` helper that assumes a fixed number of arguments instead of reading `fn.length`, breaking on functions with a different arity.
- Forgetting that `fn.length` doesn't count rest parameters or parameters with default values, which can make arity-based curry helpers behave unexpectedly for those functions.
