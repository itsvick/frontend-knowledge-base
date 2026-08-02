# Server-Components

> Subsection of 05-React.

## Overview

React Server Components (RSC) let components run and render exclusively on
the server, sending the browser a serialized description of the resulting
UI instead of JavaScript to execute — shrinking the client bundle and
letting server-only code (database calls, secrets) live directly in
components.

## Key Concepts

- **Server Components** render on the server, never ship their code to the
  client, and can't use state, effects, or browser-only APIs.
- **Client Components** (marked with `"use client"`) behave like
  traditional React components — they run in the browser, and can use
  state, effects, and event handlers.
- A tree can freely mix both — Server Components can render Client
  Components as children, passing serializable props down to them.
- **Selective hydration** lets independent parts of a Suspense-streamed
  page become interactive as soon as their own code/data is ready, without
  waiting for the rest of the page.

## Interview Questions & Answers

### Q1. Explain React Server Components.

#### Answer

React Server Components are components that render only on the server —
their code never gets sent to the browser. Instead of shipping JavaScript,
the server sends a serialized description of the rendered output, which
React uses to produce (or update) the DOM on the client. Benefits:

- **Smaller client bundles** — a Server Component's code and its
  dependencies (e.g. a heavy date-formatting or markdown library) never
  ship to the browser.
- **Direct backend access** — Server Components can query a database or
  read secrets directly, without needing a separate API layer.
- **Automatic code splitting** — since Server Component code stays on the
  server, the client only downloads what Client Components actually need.

The trade-off: Server Components can't use `useState`, `useEffect`, or
browser APIs, and can't handle events directly — interactivity requires
nesting a Client Component (`"use client"`) inside them.

### Q2. What is selective hydration?

#### Answer

Selective hydration lets React hydrate (make interactive) different parts
of a page **independently and out of order**, instead of waiting for the
entire page's JavaScript and data to be ready before hydrating anything.
When a page is split into `<Suspense>` boundaries, each boundary can
stream in and hydrate as soon as its own content is ready — and React
prioritizes hydrating whichever part the user actually interacts with
first (e.g. clicking a button in a section that hasn't hydrated yet bumps
that section ahead of others still loading). This avoids the traditional
SSR problem where the whole page's JS has to load before any of it becomes
interactive.

## Common Pitfalls

- Trying to use `useState`/`useEffect`/browser APIs inside a Server
  Component — they only work in Client Components.
- Assuming a Client Component nested inside a Server Component makes its
  *own* children Server Components too — everything below a `"use client"`
  boundary is client-rendered.
- Passing non-serializable props (functions, class instances) from a
  Server Component to a Client Component — only serializable data can
  cross that boundary.
