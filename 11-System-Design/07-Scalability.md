# Scalability

> Part of: 11-System-Design

## Overview

Scaling a frontend application to a much larger user base is less about a single fix and more about a set of complementary techniques — CDN caching, code splitting, and monitoring for client-rendered apps, plus edge caching and horizontal scaling for server-rendered ones — applied together as traffic grows.

## Key Concepts

- CDN + long-lived caching of content-hashed static assets removes JS/CSS/image delivery from the origin server entirely, regardless of traffic growth.
- Route-based and component-based code splitting keeps the initial bundle small independent of how large the app's total codebase becomes.
- CSR scaling is dominated by client-side execution cost per user (bundle size, hydration/parse time), which scales with user count independent of backend capacity.
- SSR scaling is dominated by server render cost per request, addressed via horizontal scaling of render servers, edge/CDN caching of rendered HTML, and incremental static regeneration (ISR).
- API-layer caching and a BFF (Backend-for-Frontend) reduce round-trip waterfalls and repeated backend work as request volume grows.
- Real User Monitoring (RUM) and Core Web Vitals tracking are necessary at scale to catch regressions only visible under real device/network diversity.

## Interview Questions & Answers

### Q1. How would you scale a frontend application from 1,000 to 100,000 daily users?

#### Answer

Several complementary techniques apply regardless of rendering strategy:

- **CDN + static asset caching** — serve JS/CSS/images from a CDN with long cache lifetimes. Content-hashed filenames (e.g. `main.a1b2c3.js`) make cache invalidation automatic on every deploy, so static assets never need to hit origin servers as traffic grows.
- **Code splitting / lazy loading** — route-based and component-based code splitting keeps the initial bundle small no matter how large the app's total codebase becomes. A new user at 100k scale shouldn't have to download code for features they haven't navigated to yet.
- **API layer** — introduce caching (HTTP cache headers, a CDN cache for GET-heavy endpoints, or a backend cache like Redis), and consider a BFF (Backend-for-Frontend) layer to aggregate/shape API calls so the frontend doesn't waterfall multiple round-trips as traffic — and therefore latency sensitivity — grows.
- **Monitoring** — Real User Monitoring (RUM) and Core Web Vitals tracking become necessary at scale, since regressions caused by real device/network diversity only surface once there's enough traffic to expose them; a small dev/staging environment won't catch them.

Where the two rendering approaches diverge:

- **CSR**: the bottleneck is less "backend cost" and more client-side execution cost per user — bundle size, hydration time, parse/execute time. This scales with the number of users independent of backend capacity, so bundle-size discipline (splitting, tree-shaking, deferring non-critical JS) matters more, not less, as usage grows.
- **SSR**: server-rendering cost scales roughly linearly with concurrent requests, so scaling effort shifts to backend concerns — horizontally scaling render servers, and critically, caching rendered HTML at the edge (a CDN or edge function cache keyed by URL plus relevant cache-busting params) so repeat requests for the same page don't re-render from scratch. Incremental static regeneration (ISR, as in Next.js) is a useful middle ground: pages are statically cached but can be regenerated on a schedule or on-demand without a full site rebuild.

#### Follow-up Questions

- How would you decide what cache key to use for edge-cached SSR pages that include some personalized content?
- What's the tradeoff between ISR's "stale-while-revalidate" behavior and always rendering fresh on every request?

## Common Pitfalls

- Assuming scaling is purely a backend/infra problem and ignoring client-side execution cost (bundle size, hydration time), which scales with CSR user count regardless of backend capacity.
- Caching SSR HTML at the edge without accounting for personalized/user-specific content, causing one user's page to be served to another.
- Skipping RUM/Core Web Vitals monitoring until problems are already reported by users, instead of catching regressions proactively as real-world traffic diversity increases.
