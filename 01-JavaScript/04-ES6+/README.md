# ES6+

> Subsection of 01-JavaScript.

## Overview

Modern JavaScript syntax introduced from ES6 onward: arrow functions, template literals, spread/rest, destructuring, optional chaining/nullish coalescing, classes, and ES modules.

## Key Concepts

- Arrow functions: concise syntax, no own `this`/`arguments`, can't be used as constructors
- Template literals: backtick strings with `${}` interpolation and multi-line support; tagged templates run a function over the literal's parts before it becomes a string
- Spread expands an iterable/object; rest collects arguments/properties (same `...` syntax)
- Destructuring extracts values from arrays/objects into distinct variables
- `?.` short-circuits to `undefined` at the first `null`/`undefined` link in a chain instead of throwing
- `??` (ES2020) defaults only on `null`/`undefined`, unlike `||` which defaults on any falsy value
- ES modules (`import`/`export`) are the standardized way to split code into reusable files, statically analyzable enough for bundlers to tree-shake

## Interview Questions & Answers

### Q1. What is an arrow function?

#### Answer

Arrow functions, introduced in ES6, are a shorter syntax for writing functions — declared without the `function` keyword, and usable only as a function expression, never as a declaration. Two shortcuts follow from that conciseness: if the body is a single returning expression, the `return` keyword and the curly braces `{ }` can both be omitted; and if there's exactly one parameter, the parentheses around it can be omitted too.

The other major difference — and the one that matters most in practice — is how `this` is handled. A regular function's `this` is determined by how it's called (the object before the dot at the call site). An arrow function has no `this` of its own at all — it doesn't bind one — so `this` inside it is inherited from the enclosing (lexical) scope, exactly as if `this` were just a normal variable looked up through the scope chain. This is why arrow functions can't be used as constructors and don't have their own `arguments` object either.

#### Code Example

```js
// Traditional function expression
var add = function (a, b) {
  return a + b;
};
// Arrow function expression — same result, shorter syntax
var arrowAdd = (a, b) => a + b;

// Single expression body — "return" and { } can be omitted
var arrowMultiplyBy2 = (num) => num * 2;
// Single parameter — parentheses around it can also be omitted
var arrowMultiplyBy2Short = num => num * 2;

// "this" handling — the key difference
var obj1 = {
  valueOfThis: function () {
    return this;
  },
};
var obj2 = {
  valueOfThis: () => {
    return this;
  },
};
obj1.valueOfThis(); // obj1 — this refers to the calling object
obj2.valueOfThis(); // window/global object — arrow fn inherits "this" from the enclosing scope
```

#### Follow-up Questions

- Why can't arrow functions be used as constructors with `new`?
- How does the lack of an own `arguments` object affect writing variadic arrow functions?

### Q2. What are template literals and tagged templates?

#### Answer

**Template literals** are strings written with backticks (`` ` ``) instead of quotes. The `${}` placeholder lets you interpolate any expression, not just a variable — arithmetic, ternaries, function calls — and they also support multi-line strings natively, without `\n`.

**Tagged templates** take this further: prefixing a template literal with a function name (`tag\`...\``) calls that function instead of producing a plain string. The function receives the literal's string parts as its first argument (an array split around each `${}`) and the interpolated values as the remaining arguments (conventionally gathered with a rest parameter), so it can process the static text and dynamic values separately before combining them — e.g. to escape/sanitize only the interpolated values while leaving the surrounding markup untouched.

#### Code Example

```js
// Template literals — expression interpolation and multi-line strings
const name = "Justin";
console.log(`Hello, ${name}`);

const a = 5, b = 3;
console.log(`Sum is ${a + b}`);
console.log(`Result is ${a > b ? "Yes" : "No"}`);

const text = `This is line one
This is line two`;

// Tagged template — the function runs on the literal's parts before it becomes a string
function tag(strings, ...values) {
  console.log(strings); // ["Hello ", ", you have ", " messages"]
  console.log(values);  // ["Justin", 3]
}
tag`Hello ${"Justin"}, you have ${3} messages`;

// Practical use — escape only the interpolated (user-controlled) values
function safeHTML(strings, ...values) {
  const escape = (str) => str.replace(/</g, "&lt;").replace(/>/g, "&gt;");
  return strings.reduce((result, str, i) => {
    const value = values[i] !== undefined ? escape(String(values[i])) : "";
    return result + str + value;
  }, "");
}
const userInput = "<script>alert(1)</script>";
safeHTML`<p>${userInput}</p>`; // "<p>&lt;script&gt;alert(1)&lt;/script&gt;</p>"
```

#### Follow-up Questions

- Why does a tagged template's `strings` array always have one more element than the `values` array?
- Beyond sanitizing HTML, what's another practical use of tagged templates (e.g. `styled-components`, internationalization)?

### Q3. What are the rest parameter and spread operator?

#### Answer

Both use the same `...` syntax, introduced in ES6, but do opposite things depending on where they're used.

**Rest parameter** — used in a function's parameter list to collect any number of remaining arguments into a single array, giving a cleaner way to write variadic functions. It must be the *last* parameter — `function f(a, ...args, c)` is invalid, since there'd be no unambiguous way to know where the rest ends and `c` begins.

**Spread operator** — used wherever an array, object, or set of arguments is *expected*, to expand an iterable/object into its individual elements/properties in place. Common uses: passing an array's elements as individual function arguments, shallow-cloning an array/object, and merging multiple arrays/objects together (later spreads override earlier ones on key conflicts).

In short: rest **collects** individual values into an array (used in function declarations), spread **expands** an array/object into individual values (used in function calls and array/object literals).

#### Code Example

```js
// Rest parameter — collects arguments into an array
function extractingArgs(...args) {
  return args[1];
}
extractingArgs(8, 9, 1); // 9

function addAllArgs(...args) {
  let sum = 0;
  for (let i = 0; i < args.length; i++) sum += args[i];
  return sum;
}
addAllArgs(6, 5, 7, 99); // 117

// function randomFunc(a, ...args, c) {} // SyntaxError — rest must be the last parameter
function randomFunc(a, b, ...args) {} // OK

// Spread operator — expands values in place
function addFourNumbers(num1, num2, num3, num4) {
  return num1 + num2 + num3 + num4;
}
const fourNumbers = [5, 6, 7, 8];
addFourNumbers(...fourNumbers); // 26 — spread as individual arguments

const array1 = [3, 4, 5, 6];
const clonedArray1 = [...array1]; // [3, 4, 5, 6] — shallow clone

const obj1 = { x: "Hello", y: "Bye" };
const obj2 = { z: "Yes", a: "No" };
const mergedObj = { ...obj1, ...obj2 }; // { x: "Hello", y: "Bye", z: "Yes", a: "No" }
```

#### Follow-up Questions

- Why must the rest parameter always come last in a function's parameter list?
- Why does spreading only shallow-copy, and how does that cause bugs with nested objects/arrays?

### Q4. What is destructuring?

#### Answer

Destructuring lets you extract values out of an object or array into distinct variables in a single expression, instead of accessing each property/index one at a time.

For **object destructuring**, `{ prop: newName }` extracts `prop`'s value into a variable called `newName`. If the variable should keep the same name as the property, the `: newName` part can be dropped entirely — `{ prop }` is shorthand for `{ prop: prop }`.

For **array destructuring**, `[a, b, c]` extracts elements by position — the first element into `a`, the second into `b`, and so on.

#### Code Example

```js
// Object destructuring — before ES6
const classDetails = { strength: 78, benches: 39, blackBoard: 1 };
const classStrength = classDetails.strength;
const classBenches = classDetails.benches;
const classBlackBoard = classDetails.blackBoard;

// Same thing, with destructuring
const { strength: classStrength2, benches: classBenches2, blackBoard: classBlackBoard2 } = classDetails;
console.log(classStrength2); // 78

// Shorthand when the variable name matches the property name
const { strength } = classDetails; // same as { strength: strength }
console.log(strength); // 78

// Array destructuring — before ES6
const arr = [1, 2, 3, 4];
const first = arr[0];
const second = arr[1];

// Same thing, with destructuring
const [first2, second2, third2, fourth2] = arr;
console.log(first2, second2, third2, fourth2); // 1 2 3 4

// A common real-world use: destructuring an object parameter
const person = { name: "Sam", age: 25 };
const { name: personName, age } = person;
```

#### Follow-up Questions

- How do default values work in destructuring (e.g. `const { a = 10 } = obj`)?
- How does destructuring interact with the rest parameter (e.g. `const [first, ...rest] = arr`)?

### Q5. What are optional chaining (?.) and nullish coalescing (??), and how do they differ from logical OR (||)?

#### Answer

**Optional chaining (`?.`)** safely accesses a nested property/method/index without throwing when an intermediate value is `null`/`undefined`. Instead of chaining manual checks (`user && user.profile && user.profile.address`), `user.profile.address?.city` stops and evaluates to `undefined` the moment it hits a `null`/`undefined` link, rather than throwing `Cannot read properties of undefined`. It works on property access (`obj?.a?.b`), method calls (`obj?.method()`), and array/index access (`arr?.[0]`). It's read-only — it can't be used on the left-hand side of an assignment (`user?.name = "John"` is invalid).

**Nullish coalescing (`??`)**, added in ES2020, provides a fallback value, and — together with `?.` — is commonly used to supply a default *after* an optional-chaining lookup: `user.profile?.address?.city ?? "Not Available"`. Both return the right-hand operand when the left-hand one is "empty," but they disagree on what counts as empty:

- `||` falls through on any **falsy** value: `false`, `0`, `""`, `NaN`, `null`, `undefined`.
- `??` falls through only on `null` or `undefined`, leaving other falsy values (`0`, `""`, `false`) intact.

Use `??` when a legitimate value like `0` or `""` should be kept, and `||` was traditionally used for defaulting but can incorrectly override valid falsy values. In short: optional chaining prevents errors when reading nested data, nullish coalescing supplies a fallback for the result.

#### Code Example

```js
// Optional chaining — stops safely at the first null/undefined link
const user = { profile: {} };
console.log(user.profile.address.city);  // TypeError — "address" is undefined
console.log(user.profile.address?.city); // undefined — no error

obj?.a?.b;      // property access
obj?.method();  // method call — only invoked if obj/method exist
arr?.[0];       // array access

// Nullish coalescing vs logical OR
const config = { port: 0 };
config.port || 3000; // 3000 — 0 is falsy, so || overrides a valid value
config.port ?? 3000; // 0    — 0 is not null/undefined, so ?? keeps it

// Combined — safe read + fallback
const city = user.profile?.address?.city ?? "Not Available";
```

#### Follow-up Questions

- Why can't `??` be mixed directly with `&&`/`||` in the same expression without parentheses?
- Why is `user?.name = "John"` invalid — what does that reveal about what `?.` is designed for?

### Q6. What are classes in JavaScript?

#### Answer

Classes, introduced in ES6, are syntactic sugar over constructor functions and the prototype chain (see [[01-JavaScript/03-Prototype]]) — they don't add a new inheritance model, just a cleaner syntax for the same underlying mechanism: methods defined inside a class body are still placed on the constructor's `.prototype`, exactly as if they'd been assigned manually.

Key points:
- **Not hoisted like functions** — a class can't be used before its declaration; referencing it earlier throws, the same temporal-dead-zone behavior as `let`/`const`.
- **Inheritance via `extends`** — a class can inherit another class's properties and methods.
- **Always strict mode** — the entire body of a class is implicitly `"use strict"`, regardless of the surrounding code.

#### Code Example

```js
// Before ES6 — constructor function + manually added prototype method
function Student(name, rollNumber, grade, section) {
  this.name = name;
  this.rollNumber = rollNumber;
  this.grade = grade;
  this.section = section;
}
Student.prototype.getDetails = function () {
  return `Name: ${this.name}, Roll no: ${this.rollNumber}, Grade: ${this.grade}, Section: ${this.section}`;
};
let student1 = new Student("Vivek", 354, "6th", "A");
student1.getDetails();
// "Name: Vivek, Roll no: 354, Grade: 6th, Section: A"

// ES6 class — same thing, cleaner syntax
class StudentClass {
  constructor(name, rollNumber, grade, section) {
    this.name = name;
    this.rollNumber = rollNumber;
    this.grade = grade;
    this.section = section;
  }

  getDetails() {
    return `Name: ${this.name}, Roll no: ${this.rollNumber}, Grade: ${this.grade}, Section: ${this.section}`;
  }
}
let student2 = new StudentClass("Garry", 673, "7th", "C");
student2.getDetails();
// "Name: Garry, Roll no: 673, Grade: 7th, Section: C"
```

#### Follow-up Questions

- How does `extends`/`super` work under the hood in terms of the prototype chain?
- Why does referencing a class before its declaration throw, unlike a function declaration?

### Q7. What are ES modules? Default vs named exports.

#### Answer

ES modules are the official, standardized way to package and share JavaScript code across files using `import`/`export`, so an app can be split into smaller, reusable pieces instead of one large script.

A file can export in two ways:
- **Named exports** — `export const add = ...` — a file can have multiple named exports, and each must be imported with the same name (using `{ }`) unless explicitly renamed with `as`.
- **Default export** — `export default function greet(...) {}` — only one per file, and it can be imported under any name, without `{ }`.

Beyond static `import`/`export`, **dynamic `import()`** loads a module on demand — it's asynchronous and returns a Promise, so it's used for lazy loading/code splitting rather than loading everything upfront.

**ES modules vs CommonJS** (the older Node.js module system):
- ES modules use `import`/`export` and run natively in browsers and modern Node.js; CommonJS uses `require()`/`module.exports` and was the older Node.js-only style.
- `require()` is synchronous; `import()` is asynchronous.
- Node.js treats files as ES modules when `package.json` has `"type": "module"`; browsers opt in via `<script type="module">`.

A common follow-up is **tree shaking** — since ES module `import`/`export` statements are static (resolvable without running the code), a bundler can determine exactly which exports are actually used and safely strip out the rest at build time. CommonJS's dynamic `require()` calls can't be analyzed this way, which is one reason ES modules enable smaller bundles.

#### Code Example

```js
// utils.js
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;
export default function greet(name) {
  return `Hello, ${name}`;
}

// app.js
import greet, { add, multiply } from "./utils.js";
add(2, 3);      // 5
multiply(2, 3); // 6
greet("Jasmin"); // "Hello, Jasmin"

// Renaming a named import
import { add as sum } from "./utils.js";

// Dynamic import — async, returns a Promise, good for lazy loading/code splitting
const module = await import("./utils.js");
module.add(2, 3);
```

#### Follow-up Questions

- What is tree shaking, and why does it work for ES modules but not (well) for CommonJS?
- Why might you choose a named export over a default export for a given value, or vice versa?

### Q8. What is the difference between an Object and a Map in JavaScript?

#### Answer

Both `Object` and `Map` (ES6) store key-value pairs, but they differ in several important ways:

- **Key types.** An `Object`'s keys are always coerced to strings (or `Symbol`s) — using an object as a key silently stringifies it to `"[object Object]"`. A `Map`'s keys can be *any* value — objects, functions, numbers — without any coercion, so distinct object references stay distinct keys.
- **Iteration order.** A `Map` guarantees iteration in insertion order, always. An `Object` mostly preserves insertion order too, but not fully: integer-like string keys (e.g. `"1"`, `"2"`) are reordered first, in ascending numeric order, ahead of all string keys (in insertion order) and `Symbol` keys — a quirk that can surprise you if you rely on object key order.
- **Size.** A `Map` exposes its entry count directly via `.size`. An `Object` has no built-in equivalent — you have to compute it with `Object.keys(obj).length`.
- **Iterability.** A `Map` is directly iterable — usable with `for...of` and spread (`[...map]`) out of the box. A plain `Object` isn't iterable; you need `Object.entries()`, `Object.keys()`, or `Object.values()` first to get something iterable.
- **Performance for frequent add/remove.** `Map` is optimized for scenarios with frequent addition and removal of keys. `Object` is optimized for a fixed, known shape — V8 (and similar engines) represent objects internally using "hidden classes" tied to their set of properties, and repeatedly adding/removing properties dynamically causes the engine to fall off that fast path (de-optimizing), which doesn't happen with `Map`.
- **Syntax and serialization.** A plain object gets convenient dot/bracket syntax (`obj.prop`) and, unlike `Map`, serializes directly with `JSON.stringify()`/`JSON.parse()` — `Map` isn't natively supported by `JSON.stringify()` (it serializes to `{}`) and needs manual conversion (e.g. via `Object.fromEntries()`/spreading into an array of entries) to round-trip through JSON.

In short: use `Object` for a fixed, known set of properties accessed by name (and when JSON serialization matters), and `Map` when keys are dynamic, non-string, or added/removed frequently, or when guaranteed iteration order and direct size/iterability matter.

#### Code Example

```js
// Key coercion — Object stringifies keys, Map doesn't
const objKey = {};
const obj = {};
obj[objKey] = "value";
console.log(Object.keys(obj)); // ["[object Object]"] — key was coerced to a string

const map = new Map();
map.set(objKey, "value");
console.log(map.get(objKey)); // "value" — original object reference used as-is

// Size
console.log(Object.keys(obj).length); // 1 — no direct equivalent to .size
console.log(map.size); // 1

// Iterability
for (const [key, value] of map) {
  console.log(key, value); // works directly
}
for (const [key, value] of Object.entries(obj)) {
  console.log(key, value); // Object needs Object.entries() first
}

// Integer-like keys reorder ahead of insertion order in Object, not in Map
const reordered = { b: 1, 2: "two", a: 3, 1: "one" };
console.log(Object.keys(reordered)); // ["1", "2", "b", "a"] — numeric keys first, ascending
```

#### Follow-up Questions

- Why does `JSON.stringify(new Map(...))` produce `{}`, and how would you serialize a `Map` properly?
- How would you convert between a `Map` and a plain `Object` (e.g. `Object.fromEntries(map)` and `new Map(Object.entries(obj))`)?
- Why does V8's hidden-class optimization make `Object` a poor fit for a data structure with frequently changing keys?

## Common Pitfalls

- Using an arrow function as an object method when you need `this` to refer to the object.
- Assuming spread produces a deep copy — it only shallow-copies; nested objects/arrays are still shared by reference.
- Destructuring a property that doesn't exist yields `undefined` instead of throwing, which can hide typos.
- Using `||` to default a value that can legitimately be `0`, `""`, or `false` — `??` should be used instead.
- Trying to use `?.` on the left side of an assignment (`user?.name = "John"`) — optional chaining is read-only.
- Mismatching named vs default import syntax — importing a named export without `{ }`, or a default export with `{ }`, silently gets `undefined` instead of erroring.
