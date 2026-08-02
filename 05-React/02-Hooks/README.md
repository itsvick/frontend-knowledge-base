# Hooks

> Subsection of 05-React.

## Overview

Hooks let functional components use state, side effects, and other React
features that used to require class components. This section covers the
built-in hooks beyond `useState` — effects, refs, memoization, and reducers.

## Key Concepts

- Hooks must be called at the top level of a component (not inside loops,
  conditions, or nested functions) so React can track them consistently
  across renders.
- Effect hooks (`useEffect`, `useLayoutEffect`, `useInsertionEffect`)
  synchronize a component with an external system; their dependency array
  controls when they re-run. They fire in a fixed order relative to the
  browser paint: `useInsertionEffect` → DOM mutations → `useLayoutEffect` →
  paint → `useEffect`.
- Memoization hooks (`useMemo`, `useCallback`) preserve a value/function
  reference across renders to avoid unnecessary work or re-renders — they
  are performance hints, not correctness guarantees.
- `useReducer` centralizes complex state transitions in a single reducer
  function, similar to Redux at the component level.

## Interview Questions & Answers

### Q1. Explain the useEffect lifecycle.

#### Answer

`useEffect` collapses the old class lifecycle methods
(`componentDidMount`/`componentDidUpdate`/`componentWillUnmount`) into a
single API built around an effect function and an optional cleanup
function it returns:

- **Mount** — after React commits the DOM and the browser paints, React
  runs the effect function.
- **Update** — if a dependency changed since the last render, React first
  runs the **cleanup function from the previous render**, then runs the
  effect function again with the new values.
- **Unmount** — React runs the cleanup function one final time before
  removing the component.

Effects run asynchronously after paint (so they don't block the browser
from rendering visible output), and in the order they're declared when a
component has multiple `useEffect` calls.

```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id); // cleanup: runs before next effect & on unmount
}, [tick]);
```

### Q2. What does the dependency array of useEffect affect?

#### Answer

It controls when the effect re-runs:

- **Empty array `[]`** — runs once, after the initial render only.
- **Array with variables `[a, b]`** — re-runs whenever any listed variable
  changes between renders.
- **Omitted entirely** — runs after every render.

### Q3. Why does useEffect sometimes execute twice?

#### Answer

In development, React 18+ intentionally mounts, cleans up, and re-mounts
every component once under `<StrictMode>` — which means `useEffect`
(and its cleanup) runs twice in a row. This is deliberate: it surfaces
effects that aren't properly cleaned up or aren't safe to run more than
once, before that bug reaches production. It does **not** happen in
production builds, and does not happen outside `StrictMode`.

Common effects this exposes: missing/incorrect cleanup functions,
subscriptions that leak, or effects that assume they only ever run once.
The fix is never to suppress the double-invoke — it's to make the effect
idempotent and its cleanup correct.

### Q4. What is the difference between useEffect and useLayoutEffect in React?

#### Answer

Both run side effects in function components, but at different points in
the render cycle:

- **`useEffect`** runs asynchronously, after the browser has painted —
  suitable for data fetching, subscriptions, logging, and most side
  effects.
- **`useLayoutEffect`** runs synchronously, after DOM mutations but
  *before* the browser paints — needed when you must read layout (e.g.
  measure an element) or make DOM changes that should be visible
  immediately, without a visual flicker.

### Q5. Explain useInsertionEffect.

#### Answer

`useInsertionEffect` fires **before** `useLayoutEffect`, at the point
DOM mutations happen but before any layout effects run — it exists
specifically for CSS-in-JS libraries (styled-components, emotion, etc.)
that need to inject `<style>` tags into the DOM before anything measures
layout or paints. Without it, injecting styles inside `useLayoutEffect`
could cause a layout thrash or a flash of unstyled content.

Ordering: `useInsertionEffect` → `useLayoutEffect` → `useEffect`. Refs
aren't attached yet when it runs, so it can't read layout or interact
with DOM nodes — it's meant for style injection only, and application
code should reach for `useEffect`/`useLayoutEffect` instead.

### Q6. Explain stale closures in React.

#### Answer

A stale closure happens when a function created during one render (an
effect, event handler, or callback) keeps referencing props/state
**as they were in that render**, even after newer values exist — because
the function was never recreated with the updated values. It's most
common in `useEffect`/`useCallback` when the dependency array is missing
a value the function actually uses.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count); // always logs the count from the render this effect was created in
    }, 1000);
    return () => clearInterval(id);
  }, []); // missing `count` — effect never re-runs, closure goes stale
}
```

Fixes: list the correct dependencies so the closure is recreated with
fresh values, use the functional updater form (`setCount(c => c + 1)`)
so you don't need the value at all, or store the latest value in a
`useRef` that the closure reads from.

### Q7. What is the useRef hook in React and when should it be used?

#### Answer

`useRef` returns a mutable ref object (`{ current: ... }`) that persists
across renders without causing a re-render when it changes. It's commonly
used to access a DOM node directly, or to store a mutable value (like a
timer ID or previous value) that the UI doesn't need to react to.

Use `useRef` when you need to store something across renders, but the UI
should **not** re-render when it changes.

#### Code Example

```jsx
function TextInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

### Q8. What is the difference between useRef and useState?

#### Answer

Both persist a value across renders, but they interact with rendering
completely differently:

- **`useState`** — updating it schedules a re-render, and the new value is
  reflected in that render's output. Reads/writes are tied to the render
  cycle (batched, asynchronous-feeling updates).
- **`useRef`** — mutating `.current` does **not** trigger a re-render, and
  the change is immediate/synchronous. The component won't "see" the new
  value until it re-renders for some other reason.

Rule of thumb: use `useState` for anything that should show up in the UI;
use `useRef` for values the component needs to remember but that
shouldn't, by themselves, cause the UI to update (DOM nodes, timer IDs,
previous-value tracking, mutable instance-style variables).

### Q9. What is the useCallback hook in React and when should it be used?

#### Answer

`useCallback` returns a memoized version of a function — it keeps the same
function reference between renders and only creates a new one when its
dependencies change. Use it when:

- Passing a callback down to a memoized child component (e.g. wrapped in
  `React.memo`), so the child doesn't re-render just because a new function
  instance was created.
- The child does expensive rendering.
- The function is itself a dependency of another hook (e.g. inside a
  `useEffect` dependency array).

### Q10. What is the useMemo hook in React and when should it be used?

#### Answer

`useMemo` memoizes the *result* of an expensive calculation, recomputing it
only when its dependencies change — instead of on every render. Use it for
computationally intensive derived values that don't need to be recalculated
unless their inputs actually changed.

**Internals:** React stores the value returned by the factory function
alongside the dependency array from the render that produced it. On the
next render, React compares the new dependency array to the stored one
element-by-element (using `Object.is`); if every entry matches, it returns
the cached value without calling the factory again — otherwise it re-runs
the factory and caches the new result and deps. React only keeps the
**most recent** value (not a history), and treats this cache as a
performance hint it's allowed to discard (e.g. under memory pressure), not
a semantic guarantee — so `useMemo` should never be relied on for
correctness, only for avoiding recomputation.

### Q11. When does useMemo become harmful?

#### Answer

- **Memoizing cheap computations** — comparing the dependency array and
  maintaining the cache costs more than just recomputing the value.
- **Unstable dependencies** — if a dependency is a new object/array/
  function literal created every render, the comparison never matches, so
  React recomputes anyway while still paying the memoization overhead.
- **Overuse hurts readability** — wrapping everything in `useMemo` adds
  noise and makes the code harder to follow without a measurable
  performance win, and can distract from the actual bottleneck.
- **No correctness guarantee** — because React may discard the cached
  value, code must behave correctly even if the factory re-runs more often
  than expected; never use `useMemo` to skip a side effect or guarantee a
  computation runs exactly once.

### Q12. What is the useReducer hook in React and when should it be used?

#### Answer

`useReducer` manages complex state logic as an alternative to `useState`. It
takes a reducer function `(state, action) => newState` and an initial
state, and returns the current state plus a `dispatch` function. It's a
better fit than `useState` when state has multiple related fields with
constrained update rules, or when the next state depends heavily on the
previous one.

```js
const [state, dispatch] = useReducer(reducer, initialState);
```

### Q13. What is the best practice for updating objects and arrays stored in useState?

#### Answer

Never mutate the existing object/array in place. React decides whether to
re-render by comparing the state reference (`Object.is`) to the previous
one — mutating in place leaves that reference unchanged, so React may not
detect the update at all, and any `React.memo`/`PureComponent` child relying
on shallow prop comparison will also miss the change (see [[13-Performance]]
Q1). Instead, always produce a **new** object/array and pass that to the
setter — via spread syntax, array methods that return new arrays
(`map`/`filter`/`concat`), or an immutability helper like Immer for deeply
nested state.

#### Code Example

```jsx
// Wrong — mutates existing state, same reference
setUser((prev) => {
  prev.name = "New";
  return prev;
});

// Right — new object reference
setUser((prev) => ({ ...prev, name: "New" }));

// Arrays: same idea
setItems((prev) => [...prev, newItem]); // add
setItems((prev) => prev.filter((i) => i.id !== id)); // remove
```

### Q14. Build a custom hook for API caching.

#### Answer

A custom hook for API caching wraps `fetch` in `useState`/`useEffect`,
keyed by URL, and stores completed responses in a cache that survives
across component instances (a module-level `Map`, or a ref shared via
context) so the same URL isn't refetched. It should also dedupe
in-flight requests and guard against updating state after unmount.

#### Code Example

```jsx
const cache = new Map();

function useFetchWithCache(url) {
  const [state, setState] = useState(() => {
    if (cache.has(url)) return { data: cache.get(url), loading: false, error: null };
    return { data: null, loading: true, error: null };
  });

  useEffect(() => {
    if (cache.has(url)) {
      setState({ data: cache.get(url), loading: false, error: null });
      return;
    }

    let cancelled = false;
    setState({ data: null, loading: true, error: null });

    fetch(url)
      .then((res) => res.json())
      .then((data) => {
        cache.set(url, data);
        if (!cancelled) setState({ data, loading: false, error: null });
      })
      .catch((error) => {
        if (!cancelled) setState({ data: null, loading: false, error });
      });

    return () => {
      cancelled = true;
    };
  }, [url]);

  return state;
}
```

### Q15. What is the useId hook in React and when should it be used?

#### Answer

`useId` generates a unique, stable ID for an element within a component —
useful for accessibility, e.g. linking a form `<label>` to its `<input>` via
`htmlFor`/`id`. It guarantees uniqueness across the whole app, even if the
same component renders multiple times, and is stable across server/client
rendering (unlike hand-rolled counters or `Math.random()`).

```js
const id = useId();
```

### Q16. What is the role of useSyncExternalStore in React?

#### Answer

`useSyncExternalStore` subscribes a component to state that lives **outside**
React — a browser API, or a third-party store like Redux/Zustand — in a way
that stays correct under concurrent rendering. It takes a `subscribe`
function, a `getSnapshot` function returning the current value, and
optionally a `getServerSnapshot` for SSR. Unlike a hand-rolled
`useEffect` + `useState` subscription, it guarantees the component never
reads a torn/inconsistent value mid-render if the store changes while a
concurrent render is in progress. Most app code never calls it directly —
state-management libraries use it internally to make their subscriptions
concurrent-safe.

```js
const value = useSyncExternalStore(store.subscribe, store.getSnapshot);
```

## Common Pitfalls

- Missing or incomplete dependency arrays in `useEffect`, causing stale
  closures over old props/state.
- Reaching for `useLayoutEffect` by default — it blocks painting, so it
  should only be used when `useEffect` visibly causes a flicker.
- Suppressing StrictMode's double-invoke of effects instead of fixing the
  underlying effect/cleanup so it's safe to run twice.
- Overusing `useMemo`/`useCallback` for cheap computations, adding
  complexity without a real performance win.
- Storing values in `useRef` that should actually be in state — the UI
  won't update when a ref changes.
- Reading a ref's `.current` during render and expecting it to reflect the
  latest value immediately — ref updates don't trigger or wait for a
  re-render.
