# Layout

> Part of: 04-CSS

## Overview

CSS layout governs how elements are sized and positioned on the page — through the box model that defines every element's dimensions, and positioning schemes (`static`, `relative`, `absolute`, `fixed`, `sticky`) that control where boxes end up relative to the document or viewport.

## Key Concepts

- Every element renders as a box composed of content, padding, border, and margin, from the inside out.
- `box-sizing` controls whether `width`/`height` apply to just the content box (`content-box`, the default) or to content + padding + border (`border-box`).
- `position` values (`static`, `relative`, `absolute`, `fixed`, `sticky`) determine an element's positioning scheme and containing block.
- `fixed`/`absolute` remove an element from normal document flow; `relative` keeps it in flow but offsets it visually.
- A `transform`, `filter`, `perspective`, or `will-change` on an ancestor creates a new containing block, changing what `fixed`/`absolute` descendants position relative to.

## Interview Questions & Answers

### Q1. What is the CSS Box Model?

#### Answer

Every element in CSS is rendered as a rectangular box made up of four layers, from the inside out: **content** (the actual text/images), **padding** (space between content and border), **border**, and **margin** (space outside the border, separating the element from its neighbors).

`box-sizing` controls how `width`/`height` interact with these layers:

- `content-box` (the default) — `width`/`height` apply only to the content area. Padding and border are added on top, so an element with `width: 200px; padding: 20px; border: 1px solid` actually renders at `242px` wide.
- `border-box` — `width`/`height` include padding and border, so the declared width is the element's actual final rendered width. Padding/border eat into the content area instead of adding to the total.

This is why most modern CSS resets set `box-sizing: border-box` globally — it makes sizing far more predictable, especially when combining percentage widths with padding, since nested boxes no longer overflow their parent just from adding padding.

#### Code Example

```css
* {
  box-sizing: border-box;
}

.content-box {
  box-sizing: content-box;
  width: 200px;
  padding: 20px;
  border: 1px solid black;
  /* Rendered width: 200 + 20*2 + 1*2 = 242px */
}

.border-box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 1px solid black;
  /* Rendered width: 200px (content area shrinks to fit) */
}
```

#### Follow-up Questions

- Why do percentage-based widths combined with padding cause overflow under `content-box` but not `border-box`?
- Does `box-sizing` affect margin? Why or why not?

### Q2. What does `position: fixed` mean in CSS, and what happens if `top`/`left` aren't set?

#### Answer

`position: fixed` takes an element out of normal document flow and positions it relative to the viewport rather than any ancestor, so it stays in place even as the page scrolls. This is the standard technique for sticky headers, floating action buttons, and cookie banners.

An important caveat: if any ancestor has a `transform`, `filter`, `perspective`, or `will-change` (with certain values) set, that ancestor creates a new containing block, and the `fixed` element positions relative to that ancestor instead of the viewport. This is a common source of "why isn't my fixed element actually fixed" bugs — the element then scrolls away with that ancestor instead of staying pinned to the screen.

If none of `top`/`left`/`right`/`bottom` are set, the element doesn't jump to a corner — it stays exactly where it would have naturally sat in normal flow at the moment it was removed from flow. It simply stops scrolling with the page from that visual position onward.

#### Follow-up Questions

- How does `position: sticky` differ from `fixed`, and what containing-block rules apply to it?
- Why would adding `transform: translateZ(0)` to a parent (e.g. for a GPU-accelerated animation) break a child's `fixed` positioning?

## Common Pitfalls

- Forgetting to set `box-sizing: border-box` and being surprised when padding/border push an element wider than its declared `width`.
- Assuming `fixed` always positions relative to the viewport, without checking for a `transform`/`filter`/`will-change` ancestor that silently changes its containing block.
- Assuming an element without `top`/`left` set will snap to the top-left corner once `position: fixed` is applied, when it actually stays put at its in-flow location.
