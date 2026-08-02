# Error-Boundary

> Subsection of 05-React.

## Overview

Error boundaries stop a JavaScript error in one part of the UI from
crashing the entire application, letting you show a fallback UI instead.

## Key Concepts

- Implemented as class components using `static getDerivedStateFromError`
  (to render a fallback) and `componentDidCatch` (to log the error).
- There is no Hooks equivalent yet — error boundaries must be class
  components, though they can wrap functional components.
- They only catch errors during rendering, in lifecycle methods, and in
  constructors of the tree below them — **not** in event handlers,
  asynchronous code (e.g. `setTimeout`, promises), or errors in the
  boundary itself.

## Interview Questions & Answers

### Q1. What are error boundaries in React for?

#### Answer

Error boundaries catch JavaScript errors anywhere in their child component
tree, log them, and render a fallback UI instead of letting the error crash
the whole app. They're implemented via the `componentDidCatch` and static
`getDerivedStateFromError` lifecycle methods. They do **not** catch errors
inside event handlers or asynchronous code (e.g. inside a `setTimeout` or a
promise) — those need to be handled with regular `try/catch`.

## Common Pitfalls

- Expecting an error boundary to catch errors thrown inside an `onClick` or
  other event handler — it won't; use `try/catch` there instead.
- Wrapping the entire app in a single error boundary, so any error takes
  down the whole UI instead of just the affected section.
