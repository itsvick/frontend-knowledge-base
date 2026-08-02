# Design-Patterns

> Subsection of 05-React.

## Overview

Recurring patterns for sharing behavior and building complex UIs out of
smaller, composable pieces — favoring composition over inheritance.

## Key Concepts

- **Higher-order components (HOCs)** wrap a component to inject additional
  props or behavior, without modifying the original component.
- **Composition** builds complex UIs by combining smaller components (via
  `children` or render props) rather than through class inheritance —
  React's team explicitly recommends composition over inheritance.

## Interview Questions & Answers

### Q1. What are higher-order components in React?

#### Answer

A higher-order component (HOC) is a function that takes a component as
input and returns a new component with additional props or behavior
injected — without modifying the original component. It's a pattern for
reusing component logic (e.g. `withAuth(Component)`, `connect(Component)`
from Redux).

### Q2. Explain the composition pattern in React.

#### Answer

Composition builds complex UIs by combining smaller, reusable components
rather than relying on inheritance. Instead of a component extending
another to gain behavior, it's built by passing components as `children` or
as props, letting a parent component control layout/behavior while
delegating content to whatever is passed in.

#### Code Example

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function ProfileCard() {
  return (
    <Card>
      <Avatar />
      <Bio />
    </Card>
  );
}
```

## Common Pitfalls

- Reaching for class inheritance to share behavior between components —
  React favors composition (HOCs, render props, hooks) instead.
- Stacking too many HOCs ("wrapper hell"), making the component tree hard
  to trace — custom hooks often replace this pattern more cleanly today.
