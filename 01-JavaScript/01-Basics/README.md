# Basics

> Subsection of 01-JavaScript.

## Overview

Core JavaScript fundamentals: what the language is, its data types, variable declarations, equality, coercion, hoisting, strict mode, higher-order functions, and `this`.

## Key Concepts

- High-level, interpreted, multi-paradigm language that runs in browsers and Node.js
- Primitive vs non-primitive data types; `null` vs `undefined`
- `var` (function scope) vs `let`/`const` (block scope, TDZ)
- `==` coerces types before comparing; `===` does not
- Hoisting moves declarations (not assignments) to the top of scope
- Strict mode (`"use strict"`) turns silent mistakes (accidental globals, duplicate params) into thrown errors
- Functions are first-class citizens, enabling higher-order functions (functions that take/return other functions)
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

A **shallow copy** duplicates only the top level of an object/array — nested objects/arrays are still shared by reference with the original, so mutating a nested value affects both copies. A **deep copy** recursively duplicates every nested level, so the copy shares no references with the original at any depth. This matters because JavaScript stores objects by reference, not by value — assigning or copying an object variable normally just copies the reference, not the data.

Common ways to shallow-copy: spread (`{...obj}`, `[...arr]`), `Object.assign({}, obj)`, `Array.prototype.slice()`, `Array.from()`.

Common ways to deep-copy:
- `structuredClone(obj)` — the modern built-in, handles most complex types (`Map`, `Set`, `Date`, `RegExp`, `ArrayBuffer`, circular references) correctly, but **cannot** clone functions, DOM nodes, or `Error` objects (throws a `DataCloneError`).
- `JSON.parse(JSON.stringify(obj))` — the older pattern, only safe for plain, simple data: it drops `undefined` and functions entirely, converts `Date` objects to strings, turns `Map`/`Set` into `{}`, breaks on `RegExp`, converts `Infinity`/`NaN` to `null`, and throws on circular references.
- A manual recursive clone or a library (e.g. lodash `cloneDeep`) — needed when the object contains functions/methods, since neither `structuredClone` nor JSON serialization can carry those over; objects with methods are usually application logic rather than plain data, which is why most deep-clone utilities focus on data objects and leave functions out.

In practice: use spread/`Object.assign` for flat objects, `structuredClone` for nested or complex data, and JSON serialization only for simple, fully-serializable data.

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
- What would you do if you needed a deep copy of an object that contains methods, given `structuredClone` can't clone functions?

### Q4. Difference between var, let, and const?

#### Answer

`var` has been in JavaScript since the beginning; `let` and `const` were introduced in ES2015 (ES6).

| var | let | const |
|---|---|---|
| Function scoped | Block scoped | Block scoped |
| Can be redeclared | Cannot be redeclared | Cannot be redeclared |
| Can be updated | Can be updated | Cannot be updated |

`var` is function-scoped — a variable declared with `var` anywhere inside a function is accessible throughout that whole function, regardless of which block (`if`, `for`, etc.) it was declared in. `let`/`const` are block-scoped — accessible only within the `{ }` block where they're declared.

All three are hoisted, but differently: `var` is hoisted and initialized to `undefined`, so it can be referenced (as `undefined`) before its declaration line. `let`/`const` are hoisted but left uninitialized — the span from the top of the block to the declaration line is the **temporal dead zone (TDZ)**, and referencing the variable there throws a `ReferenceError` instead of returning `undefined`.

#### Code Example

```js
function example() {
  if (true) {
    var x = 1;   // function scoped — visible outside the if block
    let y = 2;   // block scoped — only visible inside the if block
  }
  console.log(x); // 1
  // console.log(y); // ReferenceError: y is not defined
}

console.log(a); // undefined — var is hoisted and initialized
var a = 1;

console.log(b); // ReferenceError: Cannot access 'b' before initialization (TDZ)
let b = 2;

// TDZ applies inside function scope too, not just at the top level
function anotherRandomFunc() {
  console.log(message); // ReferenceError — "message" is in its TDZ here
  let message = "Hello";
}
anotherRandomFunc();
```

#### Follow-up Questions

- Why does the TDZ exist at all — what problem does it prevent that `var`'s hoisting-to-`undefined` doesn't?
- Does `typeof` avoid throwing for a variable in its TDZ, the way it does for a truly undeclared variable?

### Q5. What is the difference between == and ===?

#### Answer

- `==` compares values after type conversion.
- `===` compares both value and data type.

#### Code Example

```js
5 == "5";   // true  (string coerced to number)
5 === "5";  // false (different types)
```

### Q6. What is implicit type coercion in JavaScript?

#### Answer

Implicit type coercion is JavaScript's automatic conversion of a value from one data type to another when an expression's operands are of different types. It commonly shows up in three places: string coercion, boolean coercion, and equality coercion.

**String coercion** — happens with the `+` operator: if either operand is a string, the other operand is converted to a string and `+` concatenates instead of adding numerically (e.g. `3 + "3"` → `"33"`, `24 + "Hello"` → `"24Hello"`). Every other arithmetic operator (`-`, `*`, `/`) instead coerces string operands to numbers (e.g. `3 - "3"` → `0`).

**Boolean coercion** — happens in `if` conditions, loop checks, ternaries, and logical operators (`&&`, `||`), based on a value's "truthiness". Every value is truthy except the falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Unlike many other languages, JS's logical operators don't return `true`/`false` — they return one of the operands: `||` returns the first truthy operand (or the last operand if all are falsy); `&&` returns the first falsy operand (or the last operand if all are truthy).

**Equality coercion** — happens with `==`: both operands are converted to the same type before comparing, so `12 == "12"` is `true`. `===` performs no conversion, so `226 === "226"` is `false` since the types differ.

#### Code Example

```js
// String coercion
3 + "3";        // "33" — number coerced to string, then concatenated
24 + "Hello";   // "24Hello"
"Vivek" + " Bisht"; // "Vivek Bisht" — both strings, concatenates as usual
3 - "3";        // 0 — "-" coerces the string to a number instead

// Boolean coercion
var x = 0;
var y = 23;
if (x) { console.log(x); }  // skipped — 0 is falsy
if (y) { console.log(y); }  // runs — 23 is truthy

x = 220;
y = "Hello";
var z = undefined;
x || y;  // 220 — first operand is truthy, so it's returned
x || z;  // 220 — same reason
x && y;  // "Hello" — both truthy, so the second operand is returned
y && z;  // undefined — second operand is falsy, so it's returned

// Equality coercion
var a = 12;
var b = "12";
a == b;   // true — both coerced to the same type before comparing
a === b;  // false — no coercion, types differ ("number" vs "string")
```

#### Follow-up Questions

- What are all the falsy values in JavaScript?
- Why does `[] == false` evaluate to `true`?
- How does `==` compare `null` and `undefined` to each other and to other values?

### Q7. What is the NaN property in JavaScript?

#### Answer

`NaN` ("Not-a-Number") is a special numeric value representing the result of an operation that isn't a legal number, e.g. `0 / 0` or `parseInt("abc")`. Despite its name, `typeof NaN` is `"number"`. `NaN` is also the only value in JavaScript that is never equal to itself (`NaN === NaN` is `false`), which is why direct equality checks can't detect it.

To check whether a value is `NaN`, use `isNaN()` (or `Number.isNaN()`). The global `isNaN()` first coerces its argument to a `Number` and then checks whether the result is `NaN` — so it can return `true` for non-number values like `"Hello"` that aren't numeric after coercion, and `false` for values that coerce to a valid number (e.g. `"1"` → `1`, `true` → `1`). `Number.isNaN()` skips the coercion step and only returns `true` for the actual `NaN` value, which is usually the safer choice.

#### Code Example

```js
typeof NaN;   // "number"
NaN === NaN;  // false

isNaN("Hello");   // true — "Hello" can't be coerced to a number
isNaN(345);       // false — already a number
isNaN("1");       // false — "1" coerces to 1 (a number)
isNaN(true);      // false — true coerces to 1 (a number)
isNaN(false);     // false — false coerces to 0 (a number)
isNaN(undefined); // true — undefined coerces to NaN

Number.isNaN("Hello"); // false — no coercion, and "Hello" isn't the NaN value
Number.isNaN(NaN);     // true
```

#### Follow-up Questions

- Why is `NaN === NaN` `false`?
- How does `Number.isNaN()` differ from the global `isNaN()`?

### Q8. What is hoisting?

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

### Q9. What is strict mode in JavaScript, and what are its characteristics?

#### Answer

Strict mode is an opt-in restricted variant of JavaScript, introduced in ECMAScript 5, that makes the language throw errors for mistakes it would otherwise silently ignore — turning "silent" errors into thrown ones. This makes bugs surface earlier and makes debugging easier, since the engine no longer papers over invalid code with implicit behavior.

It's enabled by adding `"use strict";` at the top of a script or function, and is supported by all modern browsers.

Characteristics of strict mode:
- **No accidental globals** — assigning to an undeclared variable throws a `ReferenceError` instead of silently creating a global variable.
- **No duplicate parameter names** — a function can't declare two parameters with the same name.
- **Reserved words can't be used as identifiers** — words reserved for future JavaScript features (e.g. `interface`, `implements`, `package`) can't be used as variable or function names.

#### Code Example

```js
"use strict";

x = 10; // ReferenceError: x is not defined (no accidental global)

function sum(a, a) {
  // SyntaxError: Duplicate parameter name not allowed in this context
  return a + a;
}
```

#### Follow-up Questions

- How does strict mode change the value of `this` inside a regular function called without a receiver?
- Are ES6 modules and classes strict mode by default?

### Q10. What are higher-order functions in JavaScript?

#### Answer

A higher-order function is a function that either takes another function as an argument, returns a function, or both. They exist because functions in JavaScript are **first-class citizens** — they can be assigned to variables, passed as arguments, and returned from other functions, just like any other value.

Array methods like `map`, `filter`, and `reduce` are common built-in examples, since they all take a callback function as an argument.

#### Code Example

```js
// Takes a function as an argument
function higherOrder(fn) {
  fn();
}
higherOrder(function () {
  console.log("Hello world");
});

// Returns a function
function higherOrder2() {
  return function () {
    return "Do something";
  };
}
var x = higherOrder2();
x(); // "Do something"
```

#### Follow-up Questions

- How do higher-order functions relate to closures?
- What are some built-in higher-order functions in JavaScript (e.g. array methods)?

### Q11. What are callbacks in JavaScript?

#### Answer

A callback is a function passed as an argument to another function, to be executed after that other function finishes its work. Callbacks are possible because functions are first-class citizens in JavaScript — they can be passed around like any other value — which makes a callback simply the argument-passing case of a higher-order function (see Q10).

#### Code Example

```js
function divideByHalf(sum) {
  console.log(Math.floor(sum / 2));
}

function multiplyBy2(sum) {
  console.log(sum * 2);
}

function operationOnSum(num1, num2, operation) {
  var sum = num1 + num2;
  operation(sum); // the callback runs only after "sum" has been computed
}

operationOnSum(3, 3, divideByHalf);  // 3
operationOnSum(5, 5, multiplyBy2);   // 20
```

#### Follow-up Questions

- How do callbacks relate to asynchronous code (e.g. `setTimeout`, event listeners)?
- What is "callback hell", and how do Promises/`async`-`await` address it?

### Q12. What is the difference between map() and forEach()?

#### Answer

Both are array methods that take a callback and run it for each element, but they differ in intent and return value:

- **`forEach()`** — runs the callback for each element purely for its side effects (e.g. logging, updating something outside the array). It always returns `undefined`, so there's no result to work with afterward.
- **`map()`** — runs the callback for each element and builds a **new array** from the callback's return values. Use it when you need the transformed data itself, not just the side effect of iterating.

A common mistake is calling `map()` and discarding its return value — if nothing is done with the returned array, `forEach()` (or a plain `for` loop) is the more appropriate, intention-revealing choice. Also, since `map()` returns an array, it supports chaining with other array methods (`.filter()`, `.map()`, etc.); `forEach()`'s `undefined` return means it can't be chained.

#### Code Example

```js
const nums = [1, 2, 3];

// forEach — side effect only, no usable return value
const forEachResult = nums.forEach((n) => n * 2);
console.log(forEachResult); // undefined

// map — returns a new array of transformed values
const doubled = nums.map((n) => n * 2);
console.log(doubled); // [2, 4, 6]

// map supports chaining because it returns an array
const doubledThenFiltered = nums.map((n) => n * 2).filter((n) => n > 2);
console.log(doubledThenFiltered); // [4, 6]
```

#### Follow-up Questions

- Why doesn't `map()`'s callback receiving `(element, index, array)` matter for most use cases?
- When would `reduce()` be a better fit than `map()`?

### Q13. What is the difference between null and undefined?

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

### Q14. What is this in JavaScript?

#### Answer

`this` refers to the object that the currently executing function is a property of. Its value isn't fixed by where the function is defined — it depends entirely on how the function is *called* (its "receiver"). A quick rule of thumb: look at the object immediately before the dot at the call site — that's `this`. If there's no object before the dot, `this` is the global object (`window` in browsers, non-strict mode) — or `undefined` in strict mode/modules.

Because `this` is resolved at call time, not definition time, the same function can produce different `this` values depending on which object it's called through — including when a method is copied onto a different object. And since `this` is only a reference to some object, accessing a property that object doesn't actually have still throws, just like any other property access.

#### Code Example

```js
// No object before the dot — `this` is the global object
function doSomething() {
  console.log(this); // `window` in a browser (non-strict mode)
}
doSomething();

// obj is before the dot at the call site — `this` is obj
var obj = {
  name: "vivek",
  getName: function () {
    console.log(this.name);
  },
};
obj.getName(); // "vivek"

// The function is copied onto obj2 — at call time, obj2 is before the dot
var getName = obj.getName;
var obj2 = { name: "akshay", getName };
obj2.getName(); // "akshay" — not "vivek", since `this` depends on the call site

// `this` is still obj2, but obj2 has no "address" property
var obj1 = {
  address: "Mumbai, India",
  getAddress: function () {
    console.log(this.address);
  },
};
var getAddress = obj1.getAddress;
var obj3 = { name: "akshay", getAddress };
obj3.getAddress(); // undefined — `this` is obj3, which has no "address" property
```

#### Follow-up Questions

- How do arrow functions determine `this` differently from regular functions?
- How do `call`, `apply`, and `bind` let you explicitly set `this`?
- Why is `this` `undefined` instead of the global object inside a strict-mode function called without a receiver?

### Q15. Explain call(), apply(), and bind() methods.

#### Answer

`call()`, `apply()`, and `bind()` are built-in methods (available on every function) that let you explicitly set what `this` refers to when a function runs, instead of relying on the "object before the dot" at the call site.

- **`call(thisArg, arg1, arg2, ...)`** — invokes the function immediately, with `this` set to `thisArg`, passing any remaining arguments individually.
- **`apply(thisArg, [arg1, arg2, ...])`** — identical to `call()`, except the arguments are passed as a single array instead of individually.
- **`bind(thisArg, arg1, arg2, ...)`** — does *not* invoke the function. Instead it returns a **new function** with `this` permanently bound to `thisArg` (and optionally some arguments pre-filled), which can be called later, any number of times.

#### Code Example

```js
function sayHello() {
  return "Hello " + this.name;
}
var obj = { name: "Sandy" };
sayHello.call(obj); // "Hello Sandy" — borrows sayHello for obj

var person = {
  age: 23,
  getAge: function () {
    return this.age;
  },
};
var person2 = { age: 54 };
person.getAge.call(person2); // 54 — this = person2, not person

function saySomething(message) {
  return this.name + " is " + message;
}
var person4 = { name: "John" };
saySomething.call(person4, "awesome");        // "John is awesome" — args passed individually
saySomething.apply(person4, ["awesome"]);     // "John is awesome" — same, args as an array

var bikeDetails = {
  displayDetails: function (registrationNumber, brandName) {
    return this.name + ", bike details: " + registrationNumber + ", " + brandName;
  },
};
var person1 = { name: "Vivek" };
var detailsOfPerson1 = bikeDetails.displayDetails.bind(person1, "TS0122", "Bullet");
// bind() returns a new function — displayDetails hasn't run yet
detailsOfPerson1(); // "Vivek, bike details: TS0122, Bullet"
```

#### Follow-up Questions

- Why is `bind()` commonly used when passing a method as a callback (e.g. `setTimeout(obj.greet.bind(obj))`)?
- Can you `bind()` an already-bound function to a different `this`?
- How does `call`/`apply`/`bind` interact with arrow functions, which don't have their own `this`?

### Q16. What are event bubbling, capturing, and delegation? Why is delegation efficient?

#### Answer

These three are usually explained together because they all describe **event propagation** — how a user interaction (click, keypress, etc.) travels through the DOM rather than staying confined to the single element it occurred on.

**Event bubbling** is the DOM's default behavior: an event fires on the target element first, then propagates *upward* through each ancestor in turn, up to `document`. E.g. clicking an `<li>` inside a `<ul>` fires the click on the `<li>` first, then bubbles up to the `<ul>`, its parent containers, and eventually `document` — so a listener on a parent can react to clicks on any of its descendants.

**Event capturing** is the opposite: the event starts at `document`, at the *top* of the tree, and travels *downward* toward the target — before the target's own handlers run. It's enabled by passing `{ capture: true }` as `addEventListener`'s third argument. Capturing is part of the DOM event model, but most developers reach for bubbling instead, since it's more flexible (e.g. it's what makes delegation possible).

**Event delegation** is a technique that takes advantage of bubbling: instead of attaching a listener to every child element individually, you attach a *single* listener to a common parent, and inside the handler use `event.target` to figure out which child actually triggered it (often filtered with `.matches()` or `.closest()`). This is especially useful for dynamic lists, where items get added/removed after the page loads — new items are automatically covered, since the listener lives on the parent rather than on each item.

Delegation is efficient for two reasons: **fewer listeners** (one on the parent instead of N on N children, which matters at scale), and **automatic coverage of dynamically added elements** (no need to re-attach a listener to every new child).

**`event.target` vs `event.currentTarget`** — `event.target` is the element the event actually originated on; `event.currentTarget` is the element the listener is attached to. In a delegated handler on a `<ul>` where the user clicked an `<li>`, `event.target` is the `<li>` and `event.currentTarget` is the `<ul>`.

**When delegation doesn't work** — it relies on bubbling, so it only works for events that actually bubble. Events like `focus`, `blur`, and `scroll` don't bubble; for those, use their bubbling counterparts instead (`focusin`/`focusout` in place of `focus`/`blur`).

**Stopping propagation** — `event.stopPropagation()` stops the event from continuing to ancestor elements; `event.stopImmediatePropagation()` does the same but additionally prevents any other listeners registered on that *same* element from running.

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
  console.log(event.target);        // the <li> that was actually clicked
  console.log(event.currentTarget); // always the <ul> — where the listener lives
});

// Adding a new item later still works, no extra listener needed
const newItem = document.createElement("li");
newItem.textContent = "New item";
list.appendChild(newItem);

// Capturing: passing { capture: true } runs this listener on the way down
document.addEventListener(
  "click",
  () => console.log("captured at document, before the target handles it"),
  { capture: true }
);

// focus/blur don't bubble — use focusin/focusout for delegation instead
list.addEventListener("focusin", (event) => {
  console.log("focused:", event.target);
});

// Stopping propagation
list.addEventListener("click", (event) => {
  event.stopPropagation(); // parent's own click listener never fires for this event
});
```

#### Follow-up Questions

- Why do `focus`/`blur`/`scroll` not bubble, and what's the practical impact on delegation?
- When would you deliberately use capturing instead of bubbling?
- What's the difference between `stopPropagation()` and `preventDefault()`?

### Q17. What are the types of errors in JavaScript?

#### Answer

There are two broad types of errors:

- **Syntax errors** — mistakes in the code's structure (typos, mismatched brackets, invalid tokens) that prevent the code from running at all, or stop it partway through. The engine reports these with an error message pointing at the offending line.
- **Logical errors** — the syntax is valid and the code runs without throwing, but the underlying logic is wrong, so the output is incorrect. These are often harder to fix than syntax errors precisely because nothing signals that anything is wrong — there's no error to point you at the problem.

#### Code Example

```js
// Syntax error — mismatched parenthesis, code never runs
function greet( {
  console.log("Hello");
}

// Logical error — runs fine, but the wrong operator makes the result incorrect
function average(a, b) {
  return a + b / 2; // should be (a + b) / 2 — no error thrown, just a wrong answer
}
average(4, 6); // 7, not 5
```

#### Follow-up Questions

- What's a `ReferenceError` vs a `TypeError`, and when does each occur?
- How does `try`/`catch` help with runtime errors, and why can't it catch syntax errors in the same script?

## Common Pitfalls

- Using `var` inside loops with closures/callbacks, where every callback shares the same function-scoped variable.
- Assuming `const` makes objects/arrays immutable — it only prevents reassignment of the binding.
- Relying on `==` for comparisons involving `null`/`undefined`/`0`/`""`, which coerce in non-obvious ways.
- Assuming `let`/`const` aren't hoisted at all — they are, but accessing them before declaration throws (TDZ).
- Losing `this` when passing a method as a callback (e.g., `setTimeout(obj.greet)`), since it's then called without its object context.
- Assuming `&&`/`||` return `true`/`false` — they return one of the operands, not a boolean.
- Forgetting that empty containers are truthy: `if ([])` and `if ({})` both run, even though `[] == false` is `true`.
