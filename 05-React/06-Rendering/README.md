# Rendering

> Subsection of 05-React.

## Overview

Where and when a React app's HTML gets generated — on the server, at build
time, in the browser, or some hybrid of the three — has a big impact on
initial load performance, SEO, and data freshness.

## Key Concepts

- **SSR** generates HTML per-request on the server with fresh data.
- **SSG** generates HTML once at build time, served as a static file.
- **CSR** ships an empty HTML shell and renders everything client-side via
  JavaScript.
- **ISR** pre-generates pages like SSG but regenerates them in the
  background on an interval, blending SSG's speed with SSR's freshness.
- SSR typically improves perceived load time and SEO (crawlers see full
  HTML immediately) at the cost of server load and more complex hydration.

## Interview Questions & Answers

### Q1. Explain the React rendering lifecycle.

#### Answer

A React update moves through three stages:

1. **Trigger** — a render is scheduled, either from the initial mount or
   from a state/props update (`setState`, a hook setter, a parent
   re-rendering).
2. **Render phase** — React calls component functions (or `render()`) to
   figure out what the UI should look like, building a new
   work-in-progress fiber tree and diffing it against the current one.
   This phase is pure and can be paused, aborted, or restarted by React
   without any visible effect.
3. **Commit phase** — React applies the computed changes to the real DOM
   in one synchronous pass, then runs DOM-dependent side effects
   (`componentDidMount`/`componentDidUpdate`, `useLayoutEffect` for that
   pass, and `useEffect` after paint).

### Q2. What is the difference between the render phase and the commit phase?

#### Answer

| | Render phase | Commit phase |
|---|---|---|
| Purpose | Compute what changed | Apply changes to the DOM |
| Side effects | Must be pure — no DOM mutation | Where DOM mutations and effects happen |
| Interruptible? | Yes — React can pause, abort, or discard this work (concurrent rendering) | No — runs synchronously, start to finish |
| Example APIs | Component function body, `render()` | `componentDidMount`/`componentDidUpdate`, `useLayoutEffect`, `useEffect` |

Because the render phase can be thrown away or re-run before committing,
component bodies (and things like `useMemo`) must not perform side effects
— only the commit phase is guaranteed to run exactly once per completed
update.

### Q3. Explain server-side rendering of React applications and its benefits.

#### Answer

Server-side rendering (SSR) renders components to HTML on the server for
each request, then sends the fully-rendered markup to the browser, which
**hydrates** it (attaches event listeners and React's internal state)
rather than building the DOM from scratch. Benefits include faster
first-contentful-paint (the user sees content before JS even loads) and
better SEO, since crawlers receive complete HTML instead of an empty shell.

### Q4. What is SSR, SSG, CSR, ISR in React's rendering model?

#### Answer

- **SSR (Server-Side Rendering)**: HTML is generated on every request on
  the server using fresh data.
- **SSG (Static Site Generation)**: HTML is generated once at build time
  and served as a static file to every user.
- **CSR (Client-Side Rendering)**: the server sends minimal/empty HTML; the
  browser renders everything via JavaScript after the bundle loads.
- **ISR (Incremental Static Regeneration)**: HTML is pre-generated like
  SSG, but regenerated in the background at intervals — combining SSG's
  speed with SSR-like data freshness.

## Common Pitfalls

- Choosing CSR for content that needs to be indexed by search engines,
  hurting SEO.
- Assuming SSR always improves performance — it adds server load and
  hydration cost, and can hurt Time to Interactive if overused.
- Forgetting that ISR still serves stale content until the next background
  regeneration completes.
