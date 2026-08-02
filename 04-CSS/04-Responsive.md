# Responsive

> Part of: 04-CSS

## Overview

Responsive web design is the practice of building layouts that adapt to different screen sizes and devices, using fluid layouts, media queries, flexible media, and a viewport configuration that lets mobile browsers render at the device's true width.

## Key Concepts

- Fluid/relative layouts use relative units (`%`, `fr`, `em`/`rem`) instead of fixed `px`, so content reflows instead of overflowing or getting clipped.
- Media queries (`@media (max-width: ...)` / `(min-width: ...)`) apply different styles at different breakpoints.
- Mobile-first design starts with base styles for the smallest screen, then layers on complexity for larger screens via `min-width` media queries.
- Flexible images/media (`max-width: 100%; height: auto;`, plus `srcset`/`<picture>`) let images scale down and serve appropriately-sized files per viewport.
- The viewport meta tag (`<meta name="viewport" content="width=device-width, initial-scale=1">`) is required for mobile browsers to render at the device's real width instead of a zoomed-out desktop simulation.

## Interview Questions & Answers

### Q1. What are the core pillars of responsive web design?

#### Answer

Responsive design rests on a handful of complementary techniques:

- **Fluid/relative layouts** — using `%`, `fr` (in Grid), or `em`/`rem` instead of fixed `px` values so content reflows to fit its container instead of overflowing or getting clipped at different screen sizes.
- **Media queries** — `@media (max-width: ...)` / `(min-width: ...)` rules apply different styles at different breakpoints, letting layout, typography, or visibility change per screen size.
- **Mobile-first approach** — base (unqueried) styles target the smallest screen, and `min-width` media queries progressively add complexity for larger screens. This is generally preferred over desktop-first (`max-width`-driven overrides) because it forces prioritizing essential content and avoids shipping unnecessary overrides to constrained mobile devices.
- **Flexible images/media** — `img { max-width: 100%; height: auto; }` lets images scale down within their container instead of overflowing it, while `srcset`/`<picture>` let the browser choose an appropriately-sized image file per viewport/resolution instead of always downloading the largest version.
- **The viewport meta tag** — `<meta name="viewport" content="width=device-width, initial-scale=1">` is required for mobile browsers to actually render the page at the device's real CSS pixel width; without it, mobile browsers default to simulating a wide desktop viewport and zooming out, making media queries and fluid layouts effectively useless.

#### Follow-up Questions

- Why is mobile-first generally preferred over desktop-first, beyond just convention?
- What's the difference between `srcset` with width descriptors and `<picture>` with `<source>` elements — when would you use one over the other?
- How do container queries (`@container`) differ from traditional viewport-based media queries?

## Common Pitfalls

- Omitting the viewport meta tag, so media queries never trigger as expected because the mobile browser renders a zoomed-out desktop-width layout.
- Using `max-width` breakpoints as the primary strategy (desktop-first) and ending up with deeply nested overrides instead of a clean mobile-first cascade.
- Serving one large fixed-size image to all devices instead of using `srcset`/`<picture>`, wasting bandwidth on small screens.
- Setting fixed `px` widths on containers, which don't reflow and cause horizontal overflow on smaller viewports.
