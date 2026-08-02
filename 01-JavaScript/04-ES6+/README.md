# ES6+

> Subsection of 01-JavaScript.

## Overview

Modern JavaScript syntax introduced from ES6 onward: arrow functions, template literals, spread/rest, destructuring, and the nullish coalescing operator.

## Key Concepts

- Arrow functions: concise syntax, no own `this`/`arguments`, can't be used as constructors
- Template literals: backtick strings with `${}` interpolation and multi-line support
- Spread expands an iterable/object; rest collects arguments/properties (same `...` syntax)
- Destructuring extracts values from arrays/objects into distinct variables
- `??` (ES2020) defaults only on `null`/`undefined`, unlike `||` which defaults on any falsy value

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

### Q5. Difference between nullish coalescing (??) and logical OR (||)?

#### Answer

Both return the right-hand operand when the left-hand one is "empty," but they disagree on what counts as empty:

- `||` falls through on any **falsy** value: `false`, `0`, `""`, `NaN`, `null`, `undefined`.
- `??` (ES2020) falls through only on `null` or `undefined`, leaving other falsy values (`0`, `""`, `false`) intact.

Use `??` when a legitimate value like `0` or `""` should be kept, and `||` was traditionally used for defaulting but can incorrectly override valid falsy values.

#### Code Example

```js
const count = 0;

count || 10; // 10 — 0 is falsy, so || overrides it
count ?? 10; // 0  — 0 is not null/undefined, so ?? keeps it
```

#### Follow-up Questions

- Why can't `??` be mixed directly with `&&`/`||` in the same expression without parentheses?

## Common Pitfalls

- Using an arrow function as an object method when you need `this` to refer to the object.
- Assuming spread produces a deep copy — it only shallow-copies; nested objects/arrays are still shared by reference.
- Destructuring a property that doesn't exist yields `undefined` instead of throwing, which can hide typos.
- Using `||` to default a value that can legitimately be `0`, `""`, or `false` — `??` should be used instead.
