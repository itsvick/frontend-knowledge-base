# Reconciliation

> Subsection of 05-React.

## Overview

Reconciliation is how React figures out what changed between renders and
updates the real DOM efficiently, instead of re-rendering everything from
scratch. The diffing algorithm and the `key` prop are the two mechanisms
that make this both fast and correct for lists.

## Key Concepts

- On every state/props change, React builds a new virtual DOM tree and
  diffs it against the previous one to find the minimal set of changes.
- React's diffing is heuristic (O(n)), not a full tree-edit-distance
  algorithm — it assumes elements of different types produce different
  trees, and same-type elements are compared by their props.
- For lists, `key` tells React which array item an element corresponds to
  across renders, so it can match, reorder, insert, or remove items
  correctly instead of guessing by position.

## Interview Questions & Answers

### Q1. What is the diffing algorithm in React?

#### Answer

The diffing algorithm is the process React uses to compare virtual DOM
trees and compute the minimal patch to apply to the real DOM:

1. A state/props change triggers a re-render.
2. React builds a new virtual DOM tree.
3. React compares the new tree against the previous (old) virtual DOM
   tree — this comparison is the "diffing" step.
4. React calculates the minimal set of changes (a "patch").
5. Only those changes are applied to the real DOM.

#### Follow-up Questions

- How does this relate to reconciliation as a whole?

### Q2. What is Reconciliation? How does it work?

#### Answer

Reconciliation is the overall process React uses to keep the real DOM in
sync with the virtual DOM whenever state or props change. It compares the
new virtual DOM tree against the previous one (diffing), determines the
minimum number of changes required, and applies only those changes to the
actual DOM — avoiding unnecessary re-renders and keeping updates fast.

Internally, React walks both trees together, element by element, and
applies a few simple rules at each node:

- **Different element type** (e.g. `<div>` → `<span>`, or one component
  type → another) — React tears down the old subtree entirely (unmounting
  it, losing its state) and builds a brand-new one from scratch.
- **Same element type** — React keeps the same underlying DOM node/fiber
  and just updates its changed attributes/props in place, then recurses
  into its children.
- **Lists of children** — matched up using `key` rather than position, so
  reordered items are moved instead of destroyed and recreated.

### Q3. How does React determine which components should re-render?

#### Answer

By default, when a component re-renders, **all of its children re-render
too**, regardless of whether their own props changed — React doesn't
compare old vs. new props by default before deciding to render a child.
React does skip (bail out of) a subtree's re-render when:

- The component is wrapped in `React.memo` (or extends `PureComponent`)
  and a shallow comparison shows its props are unchanged.
- `shouldComponentUpdate` returns `false` in a class component.
- Nothing that could produce a different output changed — e.g. state
  updated to the same value React already had (`Object.is` bail-out).

Note that Context consumers re-render whenever the Context value changes,
**even if** they're wrapped in `React.memo` — memoization only guards
against parent-driven prop changes, not context changes.

### Q4. What is the purpose of the key prop in React?

#### Answer

`key` uniquely identifies an element within a list across renders, letting
React match old and new elements correctly during reconciliation instead of
comparing purely by position. This lets React reorder, insert, or remove
list items efficiently and correctly. Without stable, unique keys, React
may re-render or remount elements unnecessarily, causing performance issues
and subtle bugs (e.g. lost input focus/state on reorder).

### Q5. What is the consequence of using array indices as keys in React?

#### Answer

Using the array index as a `key` ties an element's identity to its
*position*, not its actual data. When the list is reordered, filtered, or
items are inserted/removed in the middle, indices shift — so React can
match the wrong element to the wrong data, causing incorrect UI updates,
lost component state (e.g. input values, focus), and unnecessary
re-renders. Index keys are only safe for lists that are static and never
reordered/filtered.

## Common Pitfalls

- Using array indices as keys for lists that can be reordered, filtered, or
  have items inserted/removed.
- Generating a new key on every render (e.g. `Math.random()`), which
  defeats the purpose of keys entirely by making every item look "new."
- Assuming reconciliation always re-renders parent and children together —
  React can bail out of re-rendering subtrees whose props/state didn't
  change.
