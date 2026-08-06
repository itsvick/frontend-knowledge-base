# Comparison

> Part of: 09-State-Management

## Overview

Choosing where state lives in a large app isn't "Redux vs Context" — it's
classifying each piece of state into one of a few buckets (local, server,
narrow cross-cutting, truly global) and matching the tool to the bucket,
rather than picking one solution for everything.

## Key Concepts

- **Colocation** — state should live as close as possible to where it's
  used; lift it only as far up the tree as the components that actually
  need it, not to a global store by default.
- **Server state vs client state** — data fetched from an API (with its
  own loading/error/staleness/caching concerns) is a different problem
  from pure UI state (a modal being open, a form's draft value); see
  [React Query](06-React-Query.md).
- **Composition over prop drilling** — passing `children`/slots down
  through layers that don't care about a value often removes the need to
  drill props through them at all, without introducing a store.
- **Context re-render cost** — every consumer of a Context re-renders on
  any change to that context's value, unless the context is split narrowly
  or paired with a selector-based store.
- **Selective subscription** — stores like Zustand/Jotai let a component
  subscribe to just the slice of global state it reads, avoiding the
  "any change re-renders everything" problem plain Context can have at
  scale.

## Interview Questions & Answers

### Q1. How do you handle state management in a large-scale app with 50+ components sharing data?

#### Answer

The real question isn't which library to reach for — it's classifying
what kind of state each shared value actually is, since each bucket has a
different right answer:

- **Local/component state** — the default, and most state in a large app
  should stay here (a single input's draft value, whether a dropdown is
  open). "50+ components share data" often overstates how much state is
  genuinely global; most of it is local state that just happens to exist
  in many places.
- **Server state** — data fetched from an API. This is a caching problem,
  not a state-management problem: a dedicated library
  ([React Query](06-React-Query.md)/SWR) handles fetching, caching,
  background revalidation, and dedup automatically. Shoving fetched data
  into Redux/Context and hand-rolling loading/error/staleness logic is
  reinventing what these libraries already solve.
- **Narrow cross-cutting state** — state shared by a specific subtree (e.g.
  a multi-step wizard's state shared by its steps, a table's shared
  sort/filter state). A `Context` scoped to just that subtree (not one
  giant context at the app root) is usually enough — or often, composition
  (passing the shared piece down as `children`/props through a wrapper
  component) avoids needing a store at all, since the "drilling" problem is
  frequently really a component-structure problem.
- **Truly global state** — state read/written unpredictably from anywhere
  in the tree (auth user, theme, feature flags, a shopping cart). Here a
  lightweight global store (Zustand/Jotai) earns its keep, specifically
  because of **selective subscription**: a component subscribes to just the
  slice of state it reads, so an unrelated state change elsewhere doesn't
  re-render it — something a single root-level Context can't do without
  manually splitting it into many smaller contexts.

The over-engineering trap is picking one tool (usually Redux) as the
default answer for "state shared across components," instead of asking
which of the four buckets a given piece of state falls into. A 50-component
app can legitimately need zero Redux/Zustand if most of its shared state is
actually server state (React Query) or narrow-subtree state
(scoped Context/composition).

#### Follow-up Questions

- How would you decide whether a shared value belongs in a scoped Context
  vs a global store like Zustand?
- How does adopting React Query for server state change how much you still
  need a global client-state store for?
- What's the actual re-render cost difference between a root-level Context
  and a selector-based store like Zustand, and when does it start to
  matter in practice?

## Common Pitfalls

- Defaulting to Redux (or any global store) for anything shared across
  more than one component, including state that's really just local or
  narrow-subtree state.
- Storing server-fetched data in a global store and hand-rolling the
  loading/error/cache-invalidation logic that React Query/SWR already
  solve.
- One giant root-level Context causing every consumer to re-render on any
  unrelated state change, instead of splitting contexts narrowly or using
  a selector-based store.
- Reaching for a global store to avoid prop drilling in a case composition
  (`children`/slots) would have solved with less coupling and no extra
  dependency.

## References

- [Kent C. Dodds: Application State Management with React](https://kentcdodds.com/blog/application-state-management-with-react)
