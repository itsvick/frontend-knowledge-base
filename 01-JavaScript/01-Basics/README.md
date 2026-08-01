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

**Answer:** JavaScript is a high-level, interpreted programming language used to create interactive web pages. It runs in browsers and on servers (Node.js).

### Q2. What are the data types in JavaScript?

**Answer:**
Primitive:
- String
- Number
- Boolean
- Undefined
- Null
- Symbol
- BigInt

Non-Primitive:
- Object
- Array
- Function

### Q3. Difference between var, let, and const?

**Answer:**

| var | let | const |
|---|---|---|
| Function scoped | Block scoped | Block scoped |
| Can be redeclared | Cannot be redeclared | Cannot be redeclared |
| Can be updated | Can be updated | Cannot be updated |

### Q4. What is the difference between == and ===?

**Answer:**
- `==` compares values after type conversion.
- `===` compares both value and data type.

### Q5. What is hoisting?

**Answer:** Hoisting is JavaScript's behavior of moving variable and function declarations to the top of their scope before execution.

### Q6. What is the difference between null and undefined?

**Answer:**
- `undefined`: Variable declared but not assigned.
- `null`: Intentional absence of a value.

### Q7. What is this in JavaScript?

**Answer:** `this` refers to the object that is currently executing the function. Its value depends on how the function is called.

## Code Examples

```js
// Q3: var vs let/const scoping
function example() {
  if (true) {
    var x = 1;   // function scoped — visible outside the if block
    let y = 2;   // block scoped — only visible inside the if block
  }
  console.log(x); // 1
  // console.log(y); // ReferenceError
}

// Q4: == vs ===
5 == "5";   // true  (string coerced to number)
5 === "5";  // false (different types)

// Q5: hoisting
console.log(a); // undefined (declaration hoisted, assignment is not)
var a = 5;

// Q6: null vs undefined
let b;
console.log(typeof b); // "undefined"
let c = null;
console.log(typeof c); // "object" (quirk of JS)

// Q7: this
const obj = {
  name: "Sam",
  greet() {
    console.log(this.name); // "Sam" — called as obj.greet()
  },
};
obj.greet();
```

## Common Pitfalls

- Using `var` inside loops with closures/callbacks, where every callback shares the same function-scoped variable.
- Assuming `const` makes objects/arrays immutable — it only prevents reassignment of the binding.
- Relying on `==` for comparisons involving `null`/`undefined`/`0`/`""`, which coerce in non-obvious ways.
- Assuming `let`/`const` aren't hoisted at all — they are, but accessing them before declaration throws (TDZ).
- Losing `this` when passing a method as a callback (e.g., `setTimeout(obj.greet)`), since it's then called without its object context.

## References

-
