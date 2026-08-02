# Prototype

> Subsection of 01-JavaScript.

## Overview

Every JavaScript object inherits properties and methods from a prototype, forming a chain the engine walks whenever a property isn't found directly on the object.

## Key Concepts

- Every object has an internal link to a prototype object it inherits from (e.g. arrays inherit from `Array.prototype`, dates from `Date.prototype`)
- `Object.prototype` sits at the top of the chain — nearly every prototype eventually inherits from it
- A prototype acts as a blueprint: properties/methods that don't exist on an object are looked up on its prototype, then that prototype's prototype, and so on

## Interview Questions & Answers

### Q1. What are object prototypes?

#### Answer

Every JavaScript object inherits properties and methods from a prototype — a "blueprint" object it's linked to. For example, `Date` objects inherit from `Date.prototype`, `Math` inherits from `Math.prototype`, and `Array` objects inherit from `Array.prototype`. At the top of this chain sits `Object.prototype`, which nearly every prototype in turn inherits from.

This is what lets you call methods that were never defined directly on an object: when the engine can't find a property/method on the object itself, it looks on the object's prototype, then that prototype's prototype, and so on up the chain — this lookup path is the **prototype chain**. If the property still isn't found by the time the chain ends (at `Object.prototype`, whose own prototype is `null`), the result is `undefined`.

#### Code Example

```js
var arr = [];
arr.push(2);
console.log(arr); // [2]

// "push" isn't defined on arr itself — the engine finds it on Array.prototype
arr.hasOwnProperty("push");        // false — not an own property of arr
Array.prototype.hasOwnProperty("push"); // true — found here instead

// hasOwnProperty itself isn't on Array.prototype either — found further up, on Object.prototype
Array.prototype.hasOwnProperty("hasOwnProperty"); // false
Object.prototype.hasOwnProperty("hasOwnProperty"); // true
```

#### Follow-up Questions

- How does `Object.create()` let you set an object's prototype directly?
- What's the difference between `__proto__` and a function's `prototype` property?
- How does prototypal inheritance differ from classical (class-based) inheritance?

### Q2. What is the Prototype design pattern?

#### Answer

The Prototype pattern creates new objects by cloning values from an existing template ("prototype") object, instead of building an uninitialized object from scratch and populating it field by field. It's especially handy when many new objects should start with the same default state — e.g. a new business/domain object initialized with the database's default settings, which live on a single prototype object that every new instance clones from.

The pattern is rarely needed in classical (class-based) languages, but it maps naturally onto JavaScript, since JS objects already inherit directly from other objects via the prototype chain (see Q1) rather than from classes — JS is a prototypal language, so cloning/inheriting from a template object is the language's native object-creation model, not an extra layer bolted on top.

#### Code Example

```js
// The prototype object — holds shared defaults
var carPrototype = {
  wheels: 4,
  drive() {
    console.log("Driving a " + this.brand);
  },
};

// New objects are created by cloning from the prototype
var car1 = Object.create(carPrototype);
car1.brand = "Toyota";

var car2 = Object.create(carPrototype);
car2.brand = "Honda";

car1.drive(); // "Driving a Toyota" — wheels/drive() come from the shared prototype
car2.wheels;  // 4 — inherited, not duplicated on each instance
```

#### Follow-up Questions

- How does `Object.create()` implement the Prototype pattern directly?
- How does the Prototype pattern compare to the Factory pattern for object creation?

### Q3. What are the different ways to create an object in JavaScript?

#### Answer

- **Object literal** — `{ }` syntax, the most common way to create a single, one-off object.
- **`new Object()`** — the `Object` constructor, functionally equivalent to a literal but more verbose.
- **Constructor function** — a regular function invoked with `new`, used as a reusable template for creating many similar objects; the function's `.prototype` becomes each instance's internal prototype (see Q1/Q2).
- **`Object.create(proto)`** — creates a new object with `proto` set directly as its prototype, the most explicit way to control the prototype chain.
- **Class** (ES6) — syntactic sugar over constructor functions and prototypes; `new ClassName()` creates an instance the same way a constructor function does, just with cleaner syntax.

Constructor functions, `Object.create()`, and classes all ultimately produce objects linked into the prototype chain — they're different syntaxes over the same underlying prototypal object-creation model.

#### Code Example

```js
// Object literal
var obj1 = { name: "Vivek" };

// Object constructor
var obj2 = new Object();
obj2.name = "Vivek";

// Constructor function
function Person(name) {
  this.name = name;
}
var obj3 = new Person("Vivek");

// Object.create — explicit prototype
var personProto = {
  greet() {
    return "Hi, " + this.name;
  },
};
var obj4 = Object.create(personProto);
obj4.name = "Vivek";

// ES6 class — sugar over constructor functions/prototypes
class PersonClass {
  constructor(name) {
    this.name = name;
  }
}
var obj5 = new PersonClass("Vivek");
```

#### Follow-up Questions

- How does a `class` compare to a constructor function under the hood?
- When would you reach for `Object.create()` over a constructor function or class?

### Q4. What is the difference between prototypal and classical inheritance?

#### Answer

**Classical inheritance** (Java, C++) is class-based: a **class** is a blueprint/generalization (e.g. `Vehicle`), and an **object** is a concrete instance of a class (e.g. a specific `Car`). Inheritance is a relationship between classes — a class extends another class, and objects only inherit indirectly, by being instances of a class somewhere in that hierarchy. Classes and objects are distinct kinds of things.

**Prototypal inheritance** (JavaScript) has no separate "class" concept underneath — objects inherit directly from other objects. Any object can serve as a prototype for another, linked via `Object.create()` or the prototype chain (see Q1), regardless of whether one is meant to "extend" the other. A prototype is simply a template object that other objects link to for properties/methods they don't have themselves.

ES6 `class` syntax (see [[01-JavaScript/04-ES6+]]) doesn't change this — it's sugar over the same prototype-linking mechanism (see Q3), not a real classical inheritance system.

#### Follow-up Questions

- Since JS only has prototypal inheritance under the hood, what does `class`/`extends` actually set up on the prototype chain?
- What's a practical downside of prototypal inheritance compared to classical inheritance (e.g. shared mutable state via a common prototype)?

## Common Pitfalls

- Confusing a function's `.prototype` property (used when it's called with `new`) with an object instance's internal prototype link (`__proto__`/`[[Prototype]]`).
- Assuming a missing property always means `undefined` outright, instead of realizing the engine may still find it further up the prototype chain.
- Assuming ES6 `class` gives JavaScript real classical inheritance — it's still prototypal inheritance underneath, just with cleaner syntax.
