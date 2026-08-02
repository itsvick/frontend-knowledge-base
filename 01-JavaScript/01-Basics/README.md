# Basics

> Subsection of 01-JavaScript.

## Overview

Core JavaScript fundamentals: what the language is, its data types, variable declarations, equality, hoisting, and `this`.

## Key Concepts

- High-level, interpreted, multi-paradigm language that runs in browsers and Node.js
- Primitive vs non-primitive data types; `null` vs `undefined`
- `var` (function scope) vs `let`/`const` (block scope, TDZ)
- `==` coerces types before comparing; `===` does not
- Hoisting moves declarations (not assignments) to the top of scope
- `this` depends on how a function is called, not where it's defined

## Interview Questions & Answers

### Q1. What is JavaScript?

#### Answer

JavaScript is a high-level, interpreted programming language used to create interactive web pages. It runs in browsers and on servers (Node.js).

### Q2. What are the data types in JavaScript?

#### Answer

JavaScript has two categories of data types: primitive and non-primitive. The `typeof` operator tells you a variable's type at runtime.

Primitive types:
- **String** — a sequence of characters, written with single or double quotes (e.g. `"Vivek"`, `'John'`).
- **Number** — represents integers and decimals (e.g. `3`, `3.6`).
- **BigInt** — stores integers beyond `Number`'s safe limit, written with an `n` suffix (e.g. `9007199254740993n`).
- **Boolean** — `true` or `false`, used for conditional logic.
- **Undefined** — the default value of a variable that's declared but not assigned.
- **Null** — represents an intentional absence of value.
- **Symbol** (ES6) — a unique and immutable value, often used as object property keys.

Non-primitive types (can hold multiple/complex values):
- **Object** — a collection of key-value pairs.
- **Array** — an ordered list of values (itself an object).
- **Function** — a callable object.

#### Code Example

```js
typeof "John Doe";      // "string"
typeof 3.14;            // "number"
typeof true;            // "boolean"
typeof 9007199254740993n; // "bigint"
typeof undefined;       // "undefined"
typeof null;            // "object" (long-standing JS bug)
typeof Symbol("id");    // "symbol"

const obj = {
  x: 43,
  y: "Hello world!",
  z: function () {
    return this.x;
  },
};

const arr = [5, "Hello", true, 4.1];
```

#### Follow-up Questions

- Why does `typeof null` return `"object"`?
- How is `BigInt` different from `Number`, and when would you need it?

### Q3. Deep copy vs shallow copy — what's the difference?

#### Answer

A **shallow copy** duplicates only the top level of an object/array — nested objects/arrays are still shared by reference with the original, so mutating a nested value affects both copies. A **deep copy** recursively duplicates every nested level, so the copy shares no references with the original at any depth.

Common ways to shallow-copy: spread (`{...obj}`, `[...arr]`), `Object.assign({}, obj)`, `Array.prototype.slice()`.

Common ways to deep-copy: `structuredClone(obj)` (modern, handles most types including circular references), `JSON.parse(JSON.stringify(obj))` (simple but drops functions/`undefined`/`Symbol`/dates become strings), or a recursive copy/library (e.g. lodash `cloneDeep`).

#### Code Example

```js
const original = { name: "Sam", address: { city: "Pune" } };

// Shallow copy — nested object is still shared
const shallow = { ...original };
shallow.address.city = "Mumbai";
console.log(original.address.city); // "Mumbai" — original mutated too

// Deep copy — fully independent
const deep = structuredClone(original);
deep.address.city = "Delhi";
console.log(original.address.city); // "Mumbai" — original untouched
```

#### Follow-up Questions

- Why does `JSON.parse(JSON.stringify(obj))` fail on objects containing functions or `Date`s?
- How does `structuredClone` handle circular references compared to `JSON.stringify`?

### Q4. Difference between var, let, and const?

#### Answer

| var | let | const |
|---|---|---|
| Function scoped | Block scoped | Block scoped |
| Can be redeclared | Cannot be redeclared | Cannot be redeclared |
| Can be updated | Can be updated | Cannot be updated |

#### Code Example

```js
function example() {
  if (true) {
    var x = 1;   // function scoped — visible outside the if block
    let y = 2;   // block scoped — only visible inside the if block
  }
  console.log(x); // 1
  // console.log(y); // ReferenceError
}
```

### Q5. What is the difference between == and ===?

#### Answer

- `==` compares values after type conversion.
- `===` compares both value and data type.

#### Code Example

```js
5 == "5";   // true  (string coerced to number)
5 === "5";  // false (different types)
```

### Q6. What is hoisting?

#### Answer

Hoisting is JavaScript's default behavior of moving variable and function declarations to the top of their scope before execution. This means that irrespective of where the variables and functions are declared, they are moved on top of the scope. The scope can be both local and global. See the code example below for `var` hoisting (global and local scope), function hoisting, and why only declarations — not initializations — are hoisted.

#### Code Example

```js
// Variable declaration hoisted above its use
hoistedVariable = 3;
console.log(hoistedVariable); // 3, even though declared after it's used
var hoistedVariable;

// Function declaration hoisted above its call
hoistedFunction(); // "Hello world!", even though declared after it's called
function hoistedFunction() {
  console.log("Hello world!");
}

// Hoisting also happens within local (function) scope
function doSomething() {
  x = 33;
  console.log(x); // 33 — "x" is hoisted inside this function's local scope
  var x;
}
doSomething();

// Only declarations are hoisted, not initializations
var a;
console.log(a); // undefined (declaration hoisted, assignment is not)
a = 5;

// Strict mode prevents implicit globals, avoiding accidental hoisting bugs
"use strict";
b = 23; // ReferenceError: b is not defined
var b;
```

### Q7. What is the difference between null and undefined?

#### Answer

- `undefined`: Variable declared but not assigned.
- `null`: Intentional absence of a value.

#### Code Example

```js
let b;
console.log(typeof b); // "undefined"
let c = null;
console.log(typeof c); // "object" (quirk of JS)
```

### Q8. What is this in JavaScript?

#### Answer

`this` refers to the object that is currently executing the function. Its value depends on how the function is called.

#### Code Example

```js
const obj = {
  name: "Sam",
  greet() {
    console.log(this.name); // "Sam" — called as obj.greet()
  },
};
obj.greet();
```

### Q9. How does event delegation work in JavaScript? Why is it efficient?

#### Answer

Event delegation relies on **event bubbling** — an event fired on a child element propagates up through its ancestors. Instead of attaching a listener to every individual child, you attach a single listener to a common parent and use `event.target` to detect which child actually triggered the event, then handle it accordingly (often filtering with `.matches()` or `.closest()`).

It's efficient for two main reasons:
- **Fewer listeners, less memory** — one listener on the parent replaces N listeners on N children, which matters when a list has hundreds/thousands of items.
- **Works for dynamically added elements** — new children added later are automatically covered, since the listener lives on the parent and doesn't need to be re-attached to every new element.

#### Code Example

```js
// Without delegation: one listener per <li>, and new items need their own listener
document.querySelectorAll("li").forEach((li) => {
  li.addEventListener("click", () => console.log(li.textContent));
});

// With delegation: one listener on the parent, handles existing and future <li>s
const list = document.querySelector("ul");
list.addEventListener("click", (event) => {
  const li = event.target.closest("li");
  if (!li || !list.contains(li)) return;
  console.log(li.textContent);
});

// Adding a new item later still works, no extra listener needed
const newItem = document.createElement("li");
newItem.textContent = "New item";
list.appendChild(newItem);
```

#### Follow-up Questions

- What's the difference between event bubbling and event capturing?
- When would delegation *not* work (e.g. events that don't bubble, like `focus`/`blur`)?
- How does `event.target` differ from `event.currentTarget` in a delegated handler?

## Common Pitfalls

- Using `var` inside loops with closures/callbacks, where every callback shares the same function-scoped variable.
- Assuming `const` makes objects/arrays immutable — it only prevents reassignment of the binding.
- Relying on `==` for comparisons involving `null`/`undefined`/`0`/`""`, which coerce in non-obvious ways.
- Assuming `let`/`const` aren't hoisted at all — they are, but accessing them before declaration throws (TDZ).
- Losing `this` when passing a method as a callback (e.g., `setTimeout(obj.greet)`), since it's then called without its object context.
