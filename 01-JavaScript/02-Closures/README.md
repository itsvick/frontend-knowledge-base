# Closures

> Subsection of 01-JavaScript.

## Overview

Scope determines where a variable or function is accessible, and the scope chain is how JavaScript looks up a variable it can't find locally. A closure is a function that remembers variables from its outer (lexical) scope even after the outer function has finished executing.

## Key Concepts

- Three kinds of scope: global, function/local, and block (`let`/`const` only — `var` ignores block scope)
- The scope chain: an unresolved variable lookup walks outward through enclosing scopes until it reaches global scope, throwing a `ReferenceError` if never found
- Formed whenever an inner function is defined inside an outer function and references the outer function's variables
- The closed-over variables persist in memory as long as the inner function is reachable
- Commonly used for data privacy, counters, and memoization
- IIFEs (Immediately Invoked Function Expressions) use closures to create a private, one-off scope

## Interview Questions & Answers

### Q1. Explain scope and the scope chain in JavaScript.

#### Answer

**Scope** determines the accessibility of variables and functions in different parts of the code — at a given point, it tells you what you can and can't reach. JavaScript has three kinds of scope:

- **Global scope** — variables/functions declared outside any function or block. Accessible from anywhere in the code.
- **Function (local) scope** — variables/functions declared inside a function. Accessible only within that function, not outside it.
- **Block scope** — applies only to `let`/`const` (not `var`). A variable declared inside a `{ }` block is accessible only within that block.

**Scope chain** is how the JavaScript engine resolves a variable reference: if a variable isn't found in the current (local) scope, the engine looks in the next outer scope, and so on, until it either finds the variable or reaches global scope. If it's not found even in global scope, a `ReferenceError` is thrown.

#### Code Example

```js
// Global scope — accessible everywhere
var globalVariable = "Hello world";
function sendMessage() {
  return globalVariable; // accessible — declared in global scope
}
function sendMessage2() {
  return sendMessage(); // accessible — sendMessage is also global
}
sendMessage2(); // "Hello world"

// Function scope — only accessible within the function
function awesomeFunction() {
  var a = 2;
  var multiplyBy2 = function () {
    console.log(a * 2); // accessible — both declared inside awesomeFunction
  };
}
// console.log(a);        // ReferenceError — a is local to awesomeFunction
// multiplyBy2();         // ReferenceError — multiplyBy2 is local to awesomeFunction

// Block scope — only let/const, not var
{
  let x = 45;
}
// console.log(x); // ReferenceError — x is scoped to the block above

for (let i = 0; i < 2; i++) {
  // do something
}
// console.log(i); // ReferenceError — i is scoped to the for loop block

// Scope chain — an unresolved lookup walks outward through enclosing scopes
var y = 24;
function favFunction() {
  var x = 667;

  var anotherFavFunction = function () {
    console.log(x); // not found locally, found in favFunction's scope — 667
  };

  var yetAnotherFavFunction = function () {
    console.log(y); // not found locally or in favFunction, found in global scope — 24
  };

  anotherFavFunction();
  yetAnotherFavFunction();
}
favFunction();
```

#### Follow-up Questions

- How does the scope chain relate to closures?
- Why does `var` ignore block scope but not function scope?

### Q2. What is a closure?

#### Answer

A closure is a function that remembers variables from its outer scope even after the outer function has finished executing. Normally, a function's local variables are discarded once it returns — but if an inner function references them, the JavaScript engine keeps those variables alive in memory for as long as that inner function is reachable, instead of destroying them. That's what lets the inner function keep reading (and updating) them long after the outer function has finished running.

This is also how closures give data privacy: a variable declared inside an outer function is never directly accessible from outside it, so exposing only an inner function that reads/writes it (e.g. via `this.getName`) creates an effectively private variable.

#### Code Example

```js
function outer() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}

const counter = outer();
counter(); // 1
counter(); // 2

// Data privacy via closure — "name" isn't accessible except through getName()
var Person = function (pName) {
  var name = pName;
  this.getName = function () {
    return name;
  };
};
var person = new Person("Neelesh");
person.getName(); // "Neelesh"
```

### Q3. What is memoization?

#### Answer

Memoization is a caching technique where a function's return value is stored (cached) based on its arguments. If the function is called again with arguments it's already seen, the cached result is returned directly instead of recomputing it — trading memory for speed. It's most useful for expensive computations that get called repeatedly with the same inputs, and closures are what make it possible: the cache needs to persist across calls without being exposed globally, which is exactly what a closure over a private variable gives you.

The tradeoff is memory: since every distinct set of arguments seen so far gets stored, memoization consumes more memory in exchange for saving computation time.

#### Code Example

```js
// Without memoization — recomputes even for a repeated argument
function addTo256(num) {
  return num + 256;
}
addTo256(20); // 276
addTo256(40); // 296
addTo256(20); // 276 — recomputed, even though we just did this

// With memoization — closure over "cache" persists across calls
function memoizedAddTo256() {
  var cache = {};
  return function (num) {
    if (num in cache) {
      console.log("cached value");
      return cache[num];
    }
    cache[num] = num + 256;
    return cache[num];
  };
}
var memoizedFunc = memoizedAddTo256();
memoizedFunc(20); // 276 — computed and cached
memoizedFunc(20); // 276 — "cached value", returned from cache
```

#### Follow-up Questions

- Why can't memoization be applied safely to functions with side effects or non-deterministic output?
- How would you memoize a function that takes multiple arguments (where a single value can't be used as the cache key)?

### Q4. What is an Immediately Invoked Function Expression (IIFE) in JavaScript?

#### Answer

An **IIFE** (pronounced "iffy") is a function that runs as soon as it's defined, instead of being called later by name. Its syntax has two parts:

```js
(function () {
  // do something
})();
```

- **The wrapping parentheses** `(function () { ... })` — when the parser sees the `function` keyword at the start of a statement, it assumes a function *declaration*, which requires a name. Without the wrapping parentheses, `function() { ... }` is a syntax error. Wrapping it in parentheses forces the parser to treat it as a function *expression* instead, which doesn't need a name.
- **The trailing parentheses** `()` — a function only runs when it's invoked. Without them, the expression just evaluates to the function itself rather than calling it. Adding `()` immediately invokes it.

IIFEs are commonly used to create a private scope — since the function's variables are closed over and inaccessible from outside — which is how the module pattern avoided polluting the global scope before ES modules and block-scoped `let`/`const` existed.

#### Code Example

```js
function () {
  // do something
}
// SyntaxError: Function statements require a function name

(function () {
  // do something
});
// Just a function expression — never invoked, nothing happens

(function () {
  console.log("Runs immediately");
})();
// Logs "Runs immediately" as soon as this line executes

// Common use: creating a private scope (module pattern)
const counter = (function () {
  let count = 0; // private — not accessible outside the IIFE
  return {
    increment: () => ++count,
  };
})();

counter.increment(); // 1
```

#### Follow-up Questions

- How does an IIFE relate to closures?
- Why were IIFEs common before `let`/`const` and ES modules existed?

## Common Pitfalls

- Creating closures over a shared `var` inside loops, so every callback captures the same final value instead of a per-iteration one (fixed by using `let`).
- Unintentionally keeping large objects alive in memory because a closure still references them.
- Forgetting the trailing `()` on an IIFE — the code parses fine but the function is never actually invoked.
