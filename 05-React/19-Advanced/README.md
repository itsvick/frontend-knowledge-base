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

### Q3. Which hooks/patterns are becoming less essential in React 19 due to the React Compiler?

#### Answer

The React Compiler (introduced with React 19) automatically memoizes
components and values at build time by statically analyzing a component's
dependencies, so manual `useMemo`, `useCallback`, and `React.memo` become
largely unnecessary for the common case of "stabilize this so a child
doesn't re-render unnecessarily" — the compiler inserts the equivalent
memoization wherever it can prove doing so is safe. Code written idiomatically
(following the Rules of React) benefits automatically, without touching the
component. It doesn't replace state/effect hooks (`useState`, `useEffect`,
`useReducer` are all still necessary) and manual memoization is still needed
for cases the compiler can't infer, such as a mutable external value the
compiler can't prove is stable — existing `useMemo`/`useCallback` calls
aren't wrong post-compiler, they just become redundant safety nets.

### Q4. What are common, standard ways to style React components?

#### Answer

- **CSS Modules** — a CSS file scoped to a single component
  (`Component.module.css`); imported class names are hashed per build,
  avoiding global class-name collisions.
- **CSS-in-JS / Styled Components** (styled-components, Emotion) — write CSS
  inside JS, colocating styles with the component and enabling styles that
  depend on props; the trade-off is added runtime cost (unless using a
  zero-runtime/compile-time variant) and a style-injection step (see
  [[02-Hooks]]'s `useInsertionEffect`).
- **Inline styles** (`style={{ ... }}`) — fully dynamic per-render styles
  with no separate stylesheet, but no pseudo-classes/media queries and no
  shared class reuse; best for one-off computed values.
- **Utility-first CSS** (Tailwind) — compose styling from small utility
  classes directly in JSX instead of a separate stylesheet per component.

There's no single "right" choice — the decision usually comes down to team
convention, whether styles need to vary per-prop, and bundle-size/runtime
trade-offs.

## Common Pitfalls

- Forgetting that a Portal's children still participate in React's context
  and event bubbling, even though they render elsewhere in the DOM.
- Not cleaning up the portal's target DOM node, leaving stale nodes behind
  after unmount.
