# ES6+

> Subsection of 01-JavaScript.

## Overview

Modern JavaScript syntax introduced from ES6 onward: arrow functions, template literals, spread/rest, and destructuring.

## Key Concepts

- Arrow functions: concise syntax, no own `this`/`arguments`, can't be used as constructors
- Template literals: backtick strings with `${}` interpolation and multi-line support
- Spread expands an iterable/object; rest collects arguments/properties (same `...` syntax)
- Destructuring extracts values from arrays/objects into distinct variables

## Interview Questions & Answers

### Q1. What is an arrow function?

#### Answer

A shorter syntax for writing functions.

#### Code Example

```js
const add = (a, b) => a + b;
```

### Q2. What are template literals?

#### Answer

Strings enclosed in backticks (`` ` ``) that support interpolation.

#### Code Example

```js
let name = "John";
console.log(`Hello ${name}`);
```

### Q3. What is the spread operator (...)?

#### Answer

It expands elements from an array or object.

#### Code Example

```js
const arr = [1, 2];
const newArr = [...arr, 3]; // [1, 2, 3]
```

### Q4. What is destructuring?

#### Answer

Extracting values from arrays or objects.

#### Code Example

```js
const person = { name: "Sam", age: 25 };
const { name: personName, age } = person;
```

## Common Pitfalls

- Using an arrow function as an object method when you need `this` to refer to the object.
- Assuming spread produces a deep copy — it only shallow-copies; nested objects/arrays are still shared by reference.
- Destructuring a property that doesn't exist yields `undefined` instead of throwing, which can hide typos.
