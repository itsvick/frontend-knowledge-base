# Concurrent

> Subsection of 05-React.

## Overview

Concurrent rendering lets React work on a UI update in the background
without blocking the main thread, so urgent updates (like a keystroke) can
interrupt and jump ahead of less urgent ones (like rendering a big list).
It's built on top of the Fiber architecture and its priority-based
Scheduler.

## Key Concepts

- Concurrent rendering makes rendering **interruptible** — React can pause
  a render, handle a more urgent update, then resume or discard the
  paused work.
- **Time slicing** is the mechanism that breaks rendering into small chunks
  so the main thread isn't blocked for one long stretch.
- APIs like `useTransition` and `startTransition` let you mark updates as
  non-urgent, so React can prioritize other, more urgent updates ahead of
  them.

## Interview Questions & Answers

### Q1. Explain concurrent rendering.

#### Answer

Concurrent rendering is a React capability (built on Fiber) where rendering
work is not treated as one uninterruptible, all-or-nothing operation.
Instead, React can start rendering an update, pause partway through if
something more urgent comes in (e.g. user input), work on the urgent update
first, and then either resume or throw away the paused render. This keeps
the UI responsive even while a large, low-priority update is in progress —
unlike React's legacy synchronous rendering, where a render always ran to
completion before yielding control back to the browser.

### Q2. What is time slicing?

#### Answer

Time slicing is the technique the Scheduler uses to implement concurrent
rendering: instead of rendering an entire tree in one long blocking pass,
React breaks the work into small units (roughly, one fiber at a time) and
periodically checks whether it still has time left in the current frame.
If not, it yields back to the browser (so it can handle input, paint,
etc.) and continues the render in a later slice — giving the appearance of
one continuous background render while never blocking the main thread for
too long at a stretch.

### Q3. What facility does the useTransition hook provide?

#### Answer

`useTransition` lets you mark a state update as a non-urgent **transition**,
so React can keep the UI responsive to more urgent updates (like a keystroke)
while the transition renders in the background. It returns
`[isPending, startTransition]` — `isPending` reflects whether the transition
is still rendering, and `startTransition` wraps the update(s) to mark as low
priority. React can interrupt a pending transition if a more urgent update
arrives, and shows the current (stale) UI rather than a fallback while it's
pending, unless the transition is combined with Suspense.

#### Code Example

```jsx
const [isPending, startTransition] = useTransition();

function handleChange(value) {
  setInputValue(value); // urgent — updates immediately
  startTransition(() => {
    setSearchQuery(value); // deprioritized — can be interrupted
  });
}
```

### Q4. What is the purpose of the useDeferredValue hook?

#### Answer

`useDeferredValue` returns a deferred copy of a value that "lags behind" the
real value during urgent updates — React renders with the previous value
first (keeping input responsive), then re-renders in the background once it
catches up to the latest value. It's the value-based counterpart to
`useTransition`: reach for `useTransition` when you control the state update
that triggers the expensive render; reach for `useDeferredValue` when you
only have the value itself (e.g. it arrives via props) and can't wrap the
update in `startTransition` yourself.

#### Code Example

```jsx
const deferredQuery = useDeferredValue(query);
// the expensive list below re-renders using deferredQuery,
// lagging behind the fast-changing `query` while typing continues smoothly
const results = useMemo(() => filterList(list, deferredQuery), [list, deferredQuery]);
```

## Common Pitfalls

- Assuming `startTransition` makes an update happen faster — it actually
  deprioritizes it, trading a slightly delayed UI update for a UI that
  stays responsive to more urgent interactions.
- Confusing concurrent rendering with multi-threading — it's all still
  single-threaded; React just interleaves smaller units of work instead of
  running one long synchronous task.
