# Advanced

> Subsection of 05-React.

## Overview

Miscellaneous advanced React topics: escaping the normal DOM hierarchy with
Portals, and common anti-patterns worth recognizing (and avoiding) in
real-world code.

## Key Concepts

- **Portals** render children into a DOM node outside the parent's DOM
  hierarchy, while keeping them inside the normal React component tree
  (context, event bubbling still work as expected).
- Anti-patterns tend to cluster around: bypassing React's data flow
  (mutating state directly), misusing lifecycle/effects, and misusing (or
  omitting) `key`.

## Interview Questions & Answers

### Q1. What are React Portals used for?

#### Answer

Portals render a component's children into a DOM node that lives outside
its parent's DOM hierarchy — while the component still behaves as if it
were nested normally in the React tree (context, event bubbling, etc. all
still work). This is useful for UI like modals, tooltips, or dropdowns that
need to escape a parent's `overflow: hidden` or `z-index` stacking context.

#### Code Example

```jsx
function Modal({ children }) {
  return ReactDOM.createPortal(children, document.getElementById("modal-root"));
}
```

### Q2. What are some React anti-patterns?

#### Answer

Common practices that lead to inefficient or hard-to-maintain React code:

- Directly mutating state instead of using `setState`/a state setter.
- Fetching data in outdated lifecycle methods like `componentWillMount`.
- Overusing `componentWillReceiveProps` instead of `componentDidUpdate` or
  derived state.
- Not using `key` in lists, or using the array index as `key`.
- Defining new inline functions/objects excessively in render, causing
  unnecessary child re-renders.
- Deeply nested state that's hard to update immutably.

## Common Pitfalls

- Forgetting that a Portal's children still participate in React's context
  and event bubbling, even though they render elsewhere in the DOM.
- Not cleaning up the portal's target DOM node, leaving stale nodes behind
  after unmount.
