# Memory-Management

> Subsection of 01-JavaScript.

## Overview

How JavaScript allocates and reclaims memory, and the APIs that work with the garbage collector rather than against it — notably `WeakMap` and `WeakSet`.

## Key Concepts

- JS uses automatic garbage collection (mark-and-sweep): memory is reclaimed once nothing reachable references it
- Regular `Map`/`Set` hold *strong* references to their keys/values, keeping them alive even if unused elsewhere
- `WeakMap`/`WeakSet` hold *weak* references, so entries can be garbage-collected once no other reference to the key exists

## Interview Questions & Answers

### Q1. Explain WeakMap and WeakSet with use cases.

#### Answer

`WeakMap` and `WeakSet` are ES6 collections similar to `Map` and `Set`, but they hold their keys (`WeakMap`) or values (`WeakSet`) **weakly** — meaning the reference doesn't prevent garbage collection. Once there's no other reference to an object used as a key/value, the entry is automatically removed and its memory reclaimed.

Key differences from `Map`/`Set`:
- Keys (`WeakMap`) / values (`WeakSet`) must be objects, not primitives.
- Not iterable — no `.keys()`, `.values()`, `.entries()`, or `size`, since the collection's contents can change at any time as the GC runs.

Common use cases:
- **Caching/memoizing data tied to an object's lifetime** — e.g. storing computed metadata for a DOM node in a `WeakMap`, without leaking memory when the node is removed.
- **Private/internal data for class instances** — using a module-scoped `WeakMap` keyed by `this` to store data inaccessible from outside the class.
- **Tracking object state without preventing cleanup** — e.g. a `WeakSet` marking which objects have already been "visited" or "processed."

#### Code Example

```js
// WeakMap: cache tied to a DOM node's lifetime
const nodeMetadata = new WeakMap();

function attachMetadata(node, data) {
  nodeMetadata.set(node, data);
}

let el = document.createElement("div");
attachMetadata(el, { clicks: 0 });

el = null; // once no other reference to el exists, its WeakMap entry
           // becomes eligible for garbage collection too

// WeakSet: tracking "already processed" objects
const processed = new WeakSet();

function process(obj) {
  if (processed.has(obj)) return;
  processed.add(obj);
  // ... do work
}
```

#### Follow-up Questions

- Why can't `WeakMap`/`WeakSet` be iterated or have their size checked?
- Why must keys be objects rather than primitives like strings or numbers?

## Common Pitfalls

- Trying to use a primitive (string, number) as a `WeakMap` key — throws a `TypeError`.
- Expecting `WeakMap`/`WeakSet` to be iterable like `Map`/`Set` — they intentionally aren't, since entries can vanish at any time.
- Reaching for `WeakMap`/`WeakSet` for general-purpose storage when a regular `Map`/`Set` (or plain object) is simpler and sufficient — weak references only matter when avoiding memory leaks tied to object lifetime.
