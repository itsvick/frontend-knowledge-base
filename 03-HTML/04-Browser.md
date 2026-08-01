# Browser

> Part of: 03-HTML

## Overview

The DOM is how the browser represents an HTML document to JavaScript, and events (with bubbling/delegation) are how the browser reports user interaction with that document.

## Key Concepts

- DOM: tree-of-objects representation of an HTML document that JS can read/manipulate
- Event bubbling: an event propagates from the target element up through its ancestors
- Event delegation: attach one listener on a parent to handle events for its (current or future) children, relying on bubbling

## Interview Questions & Answers

### Q1. What is the DOM?

**Answer:** The Document Object Model (DOM) represents an HTML document as a tree of objects that JavaScript can manipulate.

### Q2. What is event bubbling?

**Answer:** An event starts from the target element and propagates upward through its parent elements.

### Q3. What is event delegation?

**Answer:** A technique where a parent element handles events for its child elements using event bubbling.

## Code Examples

```js
// Event delegation: one listener on the parent handles clicks on any current or future <li>
document.querySelector("ul").addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log("Clicked:", e.target.textContent);
  }
});
```

## Common Pitfalls

- Attaching a listener to every child element instead of delegating to a parent, which misses dynamically added children and hurts performance.
- Forgetting `e.stopPropagation()` when a bubbling event should not reach an ancestor's handler.

## References

-
