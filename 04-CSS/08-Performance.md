# Performance

> Part of: 04-CSS

## Overview

Browser rendering performance is largely about avoiding unnecessary work in the render pipeline (style → layout → paint → composite) — especially layout (reflow) and paint (repaint), which are the most expensive stages to redo repeatedly.

## Key Concepts

- **Reflow (layout)**: the browser recalculates element geometry (size/position) — triggered by changes like width/height, adding/removing DOM nodes, or reading layout properties (`offsetWidth`, `getBoundingClientRect()`, etc.).
- **Repaint**: the browser redraws pixels for an element (color, visibility) without changing layout — cheaper than reflow, but still not free.
- **Compositing**: properties like `transform` and `opacity` can be handled on the GPU compositor thread, skipping layout and paint entirely — the cheapest path.
- Reflow and repaint are often chained: a reflow almost always triggers a repaint, but a repaint doesn't require a reflow.

## Interview Questions & Answers

### Q1. How do you prevent expensive reflows and repaints?

#### Answer

The main strategies all reduce how often the browser has to redo layout or paint work:

- **Batch DOM updates** instead of applying changes one at a time — e.g. build changes off-DOM (a `DocumentFragment` or a detached node) and attach once, or use `classList` to apply a group of style changes via one class instead of multiple inline style writes.
- **Separate layout reads from writes.** Interleaving reads (`offsetHeight`, `getBoundingClientRect()`) with writes (`style.x = ...`) inside a loop forces "layout thrashing" — each read forces the browser to flush any pending layout work synchronously to give an up-to-date value. Batch all reads first, then all writes.
- **Avoid repeated style changes inside loops** — every write to a layout-affecting property inside a loop is a potential forced synchronous reflow if a read follows it; do the minimum number of style mutations needed.
- **Prefer `transform` and `opacity` for animations** — since these can be handled purely on the compositor thread (GPU), skipping layout and paint, instead of animating properties like `top`/`left`/`width` which force reflow on every frame.
- **Take elements that will animate out of normal flow** where appropriate (e.g. `position: absolute/fixed`, or `will-change: transform`), so their changes don't force sibling elements to re-layout.

#### Code Example

```js
// Bad — layout thrashing: read, write, read, write... each read forces a synchronous reflow
els.forEach((el) => {
  const height = el.offsetHeight; // read
  el.style.height = height + 10 + "px"; // write
});

// Good — batch all reads, then all writes
const heights = els.map((el) => el.offsetHeight); // all reads first
els.forEach((el, i) => {
  el.style.height = heights[i] + 10 + "px"; // all writes after
});
```

```css
/* Bad — animating `left` triggers reflow on every frame */
.box {
  transition: left 0.3s;
}

/* Good — animating `transform` stays on the compositor, no reflow/paint */
.box {
  transition: transform 0.3s;
}
```

#### Follow-up Questions

- What's the difference between reflow and repaint, and can one happen without the other?
- Why does reading `offsetHeight` right after a style write force a synchronous reflow?
- What does `will-change` do, and what's the risk of overusing it?

## Common Pitfalls

- Reading a layout property (`offsetWidth`, `getBoundingClientRect()`) immediately after writing a style in the same loop iteration, forcing a synchronous reflow per iteration ("layout thrashing").
- Animating layout-affecting properties (`width`, `top`, `margin`) instead of `transform`, causing reflow on every animation frame.
- Overusing `will-change` on many elements, which can consume excessive GPU memory instead of improving performance.
