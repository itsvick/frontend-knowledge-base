# Core Web Vitals

> Part of: 12-Web-Performance

## Overview

Core Web Vitals (LCP, INP, CLS) are Google's user-centric metrics for
loading, interactivity, and visual stability. Bundle size and
lazy-loading strategy are two of the biggest levers for hitting good
thresholds — but the metrics only matter if they're actually monitored
against real users, not just checked once in a lab tool.

## Key Concepts

- **LCP (Largest Contentful Paint)** — good ≤2.5s, needs improvement
  2.5-4s, poor >4s. Driven by server response time, render-blocking
  resources, resource load time, and client-side rendering delay.
- **INP (Interaction to Next Paint)** — replaced FID in March 2024. Good
  ≤200ms, needs improvement 200-500ms, poor >500ms. Measures
  responsiveness across every interaction on the page's full lifecycle,
  not just the very first input.
- **CLS (Cumulative Layout Shift)** — good ≤0.1, needs improvement
  0.1-0.25, poor >0.25. Caused by images/ads/embeds without reserved
  dimensions, web-font swap causing reflow, or content injected above
  existing content.
- **RUM (field data) vs lab data** — Real User Monitoring (the `web-vitals`
  library, Chrome UX Report/CrUX) reflects actual users' devices/networks
  and is what Google's ranking signal actually uses; Lighthouse/WebPageTest
  lab data is repeatable and good for CI regression gates but doesn't
  capture real-world device/network diversity.
- **Bundle levers** — route/component-based code splitting, tree-shaking,
  dependency auditing via a bundle analyzer.
- **Lazy-loading boundaries** — lazy-load everything below the fold, but
  never the LCP candidate element itself, since delaying it directly hurts
  the metric it's supposed to help.

## Interview Questions & Answers

### Q1. How do you ensure performance at scale — Core Web Vitals, bundle size, lazy-loading strategy — and how do you make tradeoffs under deadline pressure?

#### Answer

**Thresholds.** The three Core Web Vitals each have a good / needs-improvement
/ poor band, measured at the 75th percentile of real user visits:

- **LCP** — good ≤2.5s, poor >4s.
- **INP** — good ≤200ms, poor >500ms.
- **CLS** — good ≤0.1, poor >0.25.

**Monitoring approach.** Lab tools alone aren't enough because they only
catch what they're configured to simulate:

- Ship the `web-vitals` JS library to send real user metric samples to an
  analytics backend, segmented by device tier, connection type, and route
  — a marketing landing page and an authenticated dashboard often have
  very different LCP profiles.
- Track Google Search Console / CrUX field data as ground truth, since
  that's the same real-user, 28-day rolling dataset Google's own ranking
  signal uses — a lab score that looks great can still mean a poor field
  score for slower real devices/networks.
- Run Lighthouse/WebPageTest in CI as a **regression gate** (fail a PR
  that drops projected LCP below budget) for fast feedback, while field
  data stays the actual pass/fail source of truth for whether users are
  having a good experience.
- Set explicit performance budgets tracked per release — e.g. "LCP field
  p75 ≤ 2.5s", "initial JS ≤ 170KB gzipped" — rather than just eyeballing
  scores occasionally.

**Bundle size strategy.** Route-based code splitting as the default split
boundary; dynamic `import()` for heavy, rarely-used features (a rich-text
editor, a charting library) gated behind the interaction that needs them;
a bundle analyzer (`webpack-bundle-analyzer`, `source-map-explorer`) run in
CI to catch a dependency bump that silently triples bundle size before it
ships; tree-shaking-friendly imports (`import { debounce } from 'lodash-es'`,
not the whole package).

**Lazy-loading strategy.** Lazy-load below-the-fold images/iframes
(`loading="lazy"`) and non-critical routes/components (`React.lazy`) —
but explicitly **exclude the LCP candidate** (hero image, main heading)
from any lazy-loading or code-splitting boundary that would delay it; this
is the classic mistake a naive "lazy everything" strategy makes. Preload
the LCP resource instead (`<link rel="preload" as="image" href="...">`).

**CLS specifics.** Reserve space (explicit width/height or
`aspect-ratio`) for images/embeds/ads before they load; use
`font-display: swap` paired with a size-matched fallback font (via
`size-adjust`/font-metric-matching) to reduce the reflow a swapped web
font causes; insert dynamic content (banners, cookie notices) without
shifting existing content — reserve the slot or overlay it instead of
pushing content down after it's already rendered.

**Trade-offs under deadline pressure.** Performance work under a deadline
still needs prioritization, not silent corner-cutting:

- **Triage by impact** — fix whichever metric is furthest below threshold
  on the highest-traffic route first, rather than a marginal win somewhere
  low-traffic.
- **Cheap wins first** — image format/compression (WebP/AVIF), removing
  unused JS/CSS, adding `loading="lazy"` to offscreen images: high ROI,
  low regression risk, doable same-day.
- **Defer riskier structural changes** (streaming SSR, reworking
  code-split boundaries) to a follow-up if there isn't time to properly
  test for regressions before the deadline.
- **Make the trade-off explicit**, not silent — e.g. ship with a known CLS
  issue on a low-traffic page but log it as tracked tech debt with a
  follow-up ticket, rather than letting deadline pressure quietly erode
  the whole performance budget without anyone deciding to accept that.

#### Follow-up Questions

- Why did INP replace FID as a Core Web Vital, and what does it capture
  that FID didn't?
- How would you wire an automated performance budget gate into CI, and
  what should happen when a PR fails it — block the merge, or just warn?
- If field data and lab data disagree for the same page, which do you
  trust and why?

## Common Pitfalls

- Optimizing purely against Lighthouse/lab scores without shipping RUM,
  missing how the page actually performs for real users' devices/networks.
- Lazy-loading or code-splitting the LCP element itself, directly delaying
  the metric the strategy was meant to improve.
- No explicit performance budget, so regressions creep in gradually
  without any PR being flagged as the cause.
- Treating a deadline as license to silently ship a known regression
  instead of explicitly triaging and tracking it.
- Fixing CLS by "loading things faster" instead of reserving layout space,
  which doesn't actually stop the shift once content arrives.

## References

- [web.dev: Core Web Vitals](https://web.dev/articles/vitals)
- [web.dev: INP](https://web.dev/articles/inp)
- [web.dev: web-vitals JS library](https://github.com/GoogleChrome/web-vitals)
