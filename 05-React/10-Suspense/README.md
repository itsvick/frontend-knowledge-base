# Suspense

> Subsection of 05-React.

## Overview

Suspense lets a component "pause" rendering while waiting on something
asynchronous (data or code), showing a fallback UI in the meantime. It's
the main mechanism behind lazy-loaded components and (with supporting data
libraries) async data fetching.

## Key Concepts

- Wrap a subtree in `<Suspense fallback={...}>` to show fallback content
  until everything inside has finished loading.
- `React.lazy()` pairs with Suspense to code-split components, loading
  their code on demand instead of upfront.
- Suspense only handles the "waiting" UI — the actual fetching/loading
  logic still needs to integrate with Suspense's contract (thrown
  promises), which `React.lazy` and Suspense-enabled data libraries do for
  you.

## Interview Questions & Answers

### Q1. What is React Suspense?

#### Answer

Suspense lets components "wait" for something asynchronous — like code or
data — before rendering, showing a fallback UI in the meantime instead of
blocking or showing a broken partial UI. It's commonly paired with
`React.lazy()` for code-splitting, letting a component's code load on
demand while a fallback (e.g. a spinner) displays until it's ready.

#### Code Example

```jsx
const ProfilePage = React.lazy(() => import("./ProfilePage"));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <ProfilePage />
    </Suspense>
  );
}
```

### Q2. Describe lazy loading in React, and the strategies for applying it.

#### Answer

Lazy loading defers loading a component's code until it's actually needed,
rather than including it in the initial bundle. This reduces the initial
bundle size and load time by splitting the code into smaller chunks that
load on demand. In React, it's implemented with `React.lazy()` combined
with `Suspense` to show a fallback while the chunk downloads.

The main decision is *where* to draw the split boundary — see
[[13-Performance]] for each in depth:

- **Route-based** — split by page/route, so navigating to a route fetches
  only that route's code. Usually the highest-leverage split, since most
  sessions only ever visit a subset of routes.
- **Component-level** — split an individual component within an already-
  loaded route (a modal, a rarely-opened settings panel, a heavy
  third-party widget) so it only loads once actually rendered/interacted
  with.
- **On-visibility** — defer non-code assets similarly: native `<img
  loading="lazy">` (or an `IntersectionObserver`) defers loading images
  below the fold, the same idea applied to media instead of JS chunks.

Regardless of strategy, pair the lazy boundary with an error boundary —
`React.lazy()` rejects (and throws) if the chunk fails to fetch (e.g. a
stale deploy), which without an error boundary crashes the whole subtree
instead of showing a retry/fallback UI.

### Q3. What is Suspense internally?

#### Answer

Internally, Suspense relies on a component **throwing a Promise** instead
of a value or error during render. When React renders a subtree and a
descendant throws a Promise (because its data/code isn't ready yet), React
catches it, walks up to find the nearest enclosing `<Suspense>` boundary,
and renders that boundary's `fallback` instead of the incomplete subtree.
Once the thrown Promise resolves, React re-attempts rendering the
subtree from scratch. `React.lazy()` implements this contract for code
loading; Suspense-enabled data-fetching libraries implement it for data.
This is also why plain `fetch`/`useEffect`-based data fetching does **not**
integrate with Suspense on its own — it never throws a promise for React to
catch.

## Common Pitfalls

- Forgetting to wrap a `React.lazy` component in `<Suspense>` — it will
  throw at render time without a fallback boundary.
- Lazy-loading components that are needed immediately (e.g. above-the-fold
  content), adding a loading flicker for no benefit.
- Not pairing Suspense with an error boundary — a failed lazy import needs
  an error boundary to avoid crashing the whole tree.
