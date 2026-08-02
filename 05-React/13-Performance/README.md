# Performance

> Subsection of 05-React.

## Overview

Techniques for keeping a React app fast: avoiding unnecessary re-renders
and reducing how much JavaScript the browser has to load up front.

## Key Concepts

- **Pure components** skip re-rendering when their props/state haven't
  meaningfully changed, via a shallow comparison.
- **Memoization** (`React.memo`, `useMemo`, `useCallback`) trades extra
  memory/comparison cost for skipping expensive re-renders or
  recalculations — only worth it when the thing being memoized is
  actually expensive relative to the comparison.
- **Windowing/virtualization** renders only the list items currently
  visible in the viewport (plus a small buffer), instead of the entire
  list, keeping the DOM node count constant regardless of list size.
- **Code splitting** breaks a bundle into smaller chunks loaded on demand,
  rather than shipping the entire app upfront.
- Performance work should be guided by profiling (e.g. React DevTools
  Profiler) rather than premature optimization.

## Interview Questions & Answers

### Q1. What are Pure Components?

#### Answer

Pure Components only re-render when their props or state actually change,
determined via a **shallow comparison** — preventing unnecessary
re-renders and improving performance.

- Class components opt in by extending `React.PureComponent` instead of
  `React.Component`.
- Functional components get the same effect by wrapping them in
  `React.memo`.

Because the comparison is shallow, mutating a nested object/array in place
won't be detected as a change — updates should replace objects/arrays
rather than mutate them.

### Q2. Explain `React.memo`.

#### Answer

`React.memo` is a higher-order component that memoizes a functional
component, skipping its re-render when its props are shallowly equal to
the previous render — it's the functional-component equivalent of
extending `PureComponent` (see Q1). It optionally takes a custom
comparison function `(prevProps, nextProps) => boolean` for cases where
shallow equality isn't good enough, though it's usually simpler to fix the
prop reference itself instead of reaching for a custom comparator.

It only guards against re-renders triggered by the parent — if the
component has its own state or subscribes to context that changes, it
still re-renders regardless of `React.memo`.

#### Code Example

```jsx
const Row = React.memo(
  function Row({ item }) {
    return <li>{item.label}</li>;
  },
  (prev, next) => prev.item.id === next.item.id
);
```

### Q3. `useMemo` vs `useCallback` — what's the difference?

#### Answer

Both memoize based on a dependency array, but they memoize different
things:

- `useMemo` memoizes a computed **value**, recomputing it only when its
  dependencies change.
- `useCallback` memoizes a **function reference**, returning the same
  function instance across renders unless its dependencies change.

In fact, `useCallback(fn, deps)` is equivalent to `useMemo(() => fn,
deps)`. Reach for `useMemo` for expensive derived values (e.g. filtering/
sorting a large array); reach for `useCallback` for functions passed to a
`React.memo`-wrapped child or used as a dependency of another hook, so a
new reference doesn't defeat the child's memoization or retrigger an
effect. See [[02-Hooks]] for each hook covered individually in more depth.

### Q4. What's a sound memoization strategy in a React app?

#### Answer

Memoization (`React.memo`, `useMemo`, `useCallback`) isn't free — it adds
a comparison/storage cost every render, so it only pays off when the
memoized work is genuinely expensive relative to that cost. A practical
strategy:

1. Don't memoize by default — start plain, and profile first (React
   DevTools Profiler) to find components/computations that actually
   re-render or recompute expensively.
2. Reach for `useMemo` around real computation (large-list transforms,
   derived data), not around cheap values like a formatted string.
3. Reach for `React.memo` on components that render often with the same
   props (e.g. list rows) and do non-trivial rendering work.
4. Once a child is wrapped in `React.memo`, stabilize the props it
   receives — wrap callback props in `useCallback` and object/array props
   in `useMemo`, otherwise a fresh reference every render defeats the
   memoization entirely.

### Q5. How do you optimize rendering performance in a React app?

#### Answer

Optimization is less one technique than a checklist, roughly in order of
how often each applies:

- **Avoid unnecessary re-renders**: `React.memo`/`PureComponent` (Q1, Q2)
  for components that render often with stable props; stabilize props
  with `useMemo`/`useCallback` (Q3, Q4) so memoization isn't defeated by
  new references each render; keep state as local as possible instead of
  lifting it higher than needed.
- **Reduce render cost**: memoize expensive derived computations
  (`useMemo`), and virtualize/window long lists so only visible rows
  mount (Q6, Q7).
- **Reduce work shipped up front**: code-split with `React.lazy()`/dynamic
  `import()` so the initial bundle only contains what's needed
  immediately (Q8-Q10).
- **Keep `key`s stable** in lists — an unstable `key` (e.g. array index on
  a reorderable list) forces React to unmount/remount instead of
  reconciling in place.
- **Profile before optimizing** — the React DevTools Profiler (or `why-did-you-render`,
  see Q11) should identify the actual bottleneck; optimizing a component
  that wasn't the problem just adds complexity.

### Q6. What's the difference between windowing and virtualization?

#### Answer

They're the same technique described from two angles, not two different
things: rendering only the slice of a long list that's currently visible
in the viewport (plus a small overscan buffer), instead of mounting every
item, so the DOM node count — and render cost — stays constant regardless
of how many items are in the underlying data.

- **Virtualization** is the general term for the technique (only
  materializing the visible "virtual window" of items).
- **Windowing** is the term React's ecosystem uses for the same idea,
  popularized by the `react-window`/`react-virtualized` libraries — hence
  the library name.

Without it, a list of thousands of items would mount thousands of DOM
nodes at once, tanking both initial render and scroll performance.

### Q7. Explain `react-window`.

#### Answer

`react-window` is a lightweight library that implements list/grid
virtualization (Q6) for React: it renders only the rows currently in (or
near) the viewport, recycling a small pool of DOM nodes as the user
scrolls, and positions them absolutely using calculated offsets so the
list still scrolls as if every item were present.

Key pieces:

- `FixedSizeList`/`FixedSizeGrid` — for items with a known, uniform size
  (fast, simplest offset math).
- `VariableSizeList`/`VariableSizeGrid` — for items with varying sizes,
  given an `itemSize` function.
- Each item is rendered via a render-prop function that receives `index`
  and `style` — the `style` (position/size) **must** be applied to the
  rendered element, since that's what places it correctly within the
  scrollable window.

#### Code Example

```jsx
import { FixedSizeList } from "react-window";

function Row({ index, style, data }) {
  return <div style={style}>{data[index].label}</div>;
}

function List({ items }) {
  return (
    <FixedSizeList height={400} width={300} itemCount={items.length} itemSize={35} itemData={items}>
      {Row}
    </FixedSizeList>
  );
}
```

### Q8. What is code splitting in a React application?

#### Answer

Code splitting divides an app's JavaScript into smaller chunks that load
on demand instead of all at once, reducing the initial bundle size and
load time. It's most commonly done with:

- **Dynamic `import()`** — a bundler (Webpack, Vite, etc.) treats a
  dynamic `import("./module")` call as a split point, emitting that
  module as a separate chunk fetched only when the call runs. It returns
  a Promise resolving to the module, so it can be called conditionally or
  in response to an event, not just at the top of a file.
- **`React.lazy()` + `Suspense`** — wraps a dynamic import so its result
  can be rendered as a component, showing a `fallback` while the chunk
  downloads (see [[10-Suspense]] for how this works internally).

Splitting is typically applied at two granularities — by route (Q9) and
by component (Q10).

### Q9. Explain route-based code splitting.

#### Answer

Route-based code splitting loads each route's code only when the user
navigates to it, rather than bundling every page/route into the initial
load. It's the highest-leverage place to split, since a user visiting one
route doesn't need the JavaScript for every other route up front.

```jsx
const Home = React.lazy(() => import("./routes/Home"));
const Settings = React.lazy(() => import("./routes/Settings"));

function App() {
  return (
    <Suspense fallback={<PageSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### Q10. Explain component-level code splitting.

#### Answer

Component-level code splitting defers loading an individual component's
code — rather than a whole route — until it's actually needed, using the
same `React.lazy()` + `Suspense` mechanism at a finer grain. It's used for
components that are expensive but not always needed on a given route:

- Content gated behind user interaction (a modal, a dialog opened on
  click) that most visitors never trigger.
- Heavy, rarely-used widgets (a rich text editor, a charting library)
  where most of a page doesn't need them immediately.
- Below-the-fold sections not needed for the initial paint.

The tradeoff versus route-based splitting is granularity: it avoids
loading code that a given session may never use, at the cost of an extra
network request (and a fallback UI) the first time that component
mounts.

### Q11. How would you debug unnecessary renders in a React app?

#### Answer

- **React DevTools Profiler** — record a session and look at which
  components re-rendered and why; the "Ranked"/"Flamegraph" views and the
  "why did this render" info (highlighting changed props/state/hooks) are
  the fastest way to find the culprit.
- **Highlight updates when components render** (a DevTools setting) —
  visually flashes components on every render, useful for spotting a
  component re-rendering far more than expected.
- **`why-did-you-render`** (library) — patches React to log the specific
  prop/state change that caused a re-render, including cases where a prop
  looks equal but is a new reference.
- Once found, check for the usual causes: a new object/array/function
  literal passed as a prop every render, missing `React.memo` on a child
  that receives stable props, context value objects recreated every
  render (re-renders every consumer), or state lifted higher than it
  needs to be.

## Common Pitfalls

- Wrapping every component in `React.memo`/`PureComponent` regardless of
  whether it actually re-renders expensively — the shallow-comparison
  overhead can outweigh the benefit.
- Mutating props/state in place, which defeats shallow comparison since the
  reference doesn't change.
- Wrapping a child in `React.memo` but still passing it a new inline
  object/array/function literal every render — the memoization never
  triggers because the prop reference always changes.
- Overusing `useMemo`/`useCallback` on cheap values, adding complexity
  without a measurable win — see [[02-Hooks]] for the same pitfall from the
  hooks' side.
- Virtualizing a list without giving `react-window` accurate item
  sizes (for `VariableSizeList`), causing visible jumps/overlaps as it
  corrects its offset estimates while scrolling.
- Code-splitting so aggressively that the UI shows loading spinners for
  content needed immediately.
- Lazy-loading a component without wrapping it in an error boundary — a
  failed chunk fetch (e.g. a stale deploy) crashes the tree instead of
  showing a retry/fallback UI.
