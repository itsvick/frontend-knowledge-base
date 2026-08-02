# Fiber

> Subsection of 05-React.

## Overview

Fiber is the internal reconciliation engine React has used since v16,
replacing the older synchronous "stack" reconciler. It's what makes
concurrent features (like Suspense and time-slicing) possible.

## Key Concepts

- Fiber breaks rendering work into small units ("fibers," one per
  component) that can be paused, resumed, aborted, or prioritized.
- This enables React to yield back to the browser between chunks of work,
  keeping the UI responsive even during large updates.
- It's the foundation for concurrent features: error boundaries, Suspense,
  and priority-based scheduling of updates.

## Interview Questions & Answers

### Q1. What is React Fiber?

#### Answer

React Fiber is a complete rewrite of React's core reconciliation algorithm
(introduced in React 16), designed to improve rendering performance and
enable features like incremental rendering, async rendering, and error
boundaries. Instead of processing an entire update synchronously in one
go, Fiber breaks the render process into small units of work that React
can pause, resume, abort, or prioritize — so a large update doesn't block
the main thread and freeze the UI.

### Q2. Explain how React Fiber works internally.

#### Answer

Each component instance is represented by a **fiber node** — a plain JS
object holding its type, props, state, and pointers to its parent, first
child, and next sibling, forming a linked-list tree (rather than a
recursive call stack) that can be walked incrementally and paused midway.

React keeps **two fiber trees**:

- The **current tree** — reflects what's already committed to the DOM.
- The **work-in-progress tree** — built during a render, node by node,
  based on the incoming update.

React processes fiber nodes in a **work loop**: it works on one fiber, then
checks if there's time left in the current frame (yielding to the browser
if not) before moving to the next. Once the whole work-in-progress tree is
complete, React **commits** it in one synchronous pass — swapping it in as
the new current tree (this "double buffering" is why partial/incomplete
work is never shown to the user).

### Q3. What is the React Scheduler?

#### Answer

The Scheduler is the package that decides **when** and in what **order**
React processes units of fiber work. It assigns each update a priority
(e.g. a user click is more urgent than a background data fetch), and uses
the browser's idle time (cooperatively yielding via APIs conceptually
similar to `requestIdleCallback`) to fit rendering work into small slices
without blocking user input — this priority-based, interruptible scheduling
is what enables concurrent features like transitions and time slicing.

## Common Pitfalls

- Assuming all React updates are synchronous — Fiber allows React to
  interrupt and reprioritize rendering work.
- Confusing Fiber (the internal reconciler) with "concurrent features" —
  Fiber is the underlying architecture that makes those features possible,
  not a feature itself.
