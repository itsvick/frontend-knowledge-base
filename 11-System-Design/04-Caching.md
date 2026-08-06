# Caching

> Part of: 11-System-Design

## Overview

Client-side caching stores fetched data locally to avoid redundant network
requests, trading off freshness against speed. For frequently-changing
data, the design question isn't "cache or don't" — it's how to serve a
cached value instantly while keeping it from silently going stale forever.

## Key Concepts

- **Stale-while-revalidate (SWR)** — serve the cached (possibly stale)
  value immediately, kick off a background refetch, then swap in the fresh
  value when it lands. Gives an instant perceived load without sacrificing
  eventual freshness.
- **Cache invalidation strategies** — time-based (TTL/`staleTime`),
  mutation-triggered (invalidate a specific key right after a mutation that
  changes it), and event-driven (a WebSocket/SSE push tells the client
  "this changed, refetch").
- **Cache key design** — the key must include every input that affects the
  response (endpoint + serialized params/filters/user id); too broad a key
  causes different requests to collide on one cached entry, too narrow
  defeats caching entirely.
- **Garbage collection** — evict entries no component has observed in a
  while (`cacheTime`/`gcTime`), so a long session's cache doesn't grow
  unbounded.
- **When not to cache** — data where staleness has real consequences
  (checkout pricing/inventory), one-off queries unlikely to repeat, and
  mutation responses (never treat a POST/PUT result as a cacheable GET).

## Interview Questions & Answers

### Q1. How would you design client-side caching for an app with frequently changing data?

#### Answer

**Core pattern: stale-while-revalidate.** Show the cached value instantly
if one exists (no loading spinner), trigger a background refetch, and
silently swap in the fresh value when it arrives. This is what
[React Query](../09-State-Management/06-React-Query.md)/SWR do by default
via `staleTime`/`cacheTime` config, and it's the right default for data
that changes often but where showing a slightly-stale value for a moment
is acceptable.

**Cache invalidation strategy** — pick the mechanism per data source, not
one global rule:

- **Time-based (`staleTime`)** for data that changes on a roughly
  predictable cadence — cheap, no coordination needed, but can serve
  stale data within the window.
- **Mutation-triggered invalidation** for anything the user's own action
  changes — e.g. after a "post comment" mutation succeeds, explicitly
  invalidate the `comments` query key so the next read refetches
  immediately, instead of waiting out a TTL.
- **Event-driven invalidation** for data changed by other users/systems —
  a WebSocket/SSE message ("this resource changed") triggers a targeted
  refetch or in-place cache patch instead of polling to detect changes.

**Cache key granularity.** Key by exactly the inputs that affect the
response: endpoint plus every filter/pagination/param that changes the
result. Too broad a key (e.g. keying only by endpoint, ignoring filters)
makes a filtered view and an unfiltered view collide and show the wrong
data; too narrow a key (e.g. including a cache-busting timestamp) means
nothing ever hits the cache at all.

**When NOT to cache:**

- Data where staleness has real consequences — checkout inventory/pricing,
  account balances — read through to the server (or use a near-zero TTL)
  instead of trusting a cache.
- Highly personalized or genuinely one-off queries unlikely to be repeated
  soon — the caching machinery adds complexity and memory for no real
  hit-rate benefit.
- Mutation responses (POST/PUT/DELETE) — treat them as one-time results
  that trigger invalidation of related reads, not as cacheable entries
  themselves.

Finally, garbage-collect entries no mounted component is observing after
some idle period, so a long-lived session's cache doesn't grow unbounded.

#### Follow-up Questions

- How would you invalidate a cached list when a single item in it changes,
  without refetching the whole list?
- How does stale-while-revalidate interact with an optimistic update —
  what happens if the background revalidation disagrees with the
  optimistic value?
- At what point does event-driven invalidation (via WebSocket) become
  worth the added infrastructure over just a shorter TTL?

## Common Pitfalls

- Caching data (checkout pricing, inventory, balances) where staleness has
  real consequences, instead of reading through or using a near-zero TTL.
- Cache keys that don't include all the params that affect the response,
  causing a filtered view to show another view's cached data.
- Relying purely on TTL expiry for data the user's own mutations change,
  instead of explicitly invalidating the affected key right after the
  mutation succeeds.
- Treating a mutation's response as if it were a cacheable GET result.
- No garbage collection, letting the cache grow unbounded over a long
  session.

## References

- [TanStack Query docs: Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
