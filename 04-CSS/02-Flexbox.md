# Flexbox

> Part of: 04-CSS

## Overview

Flexbox is a one-dimensional CSS layout model for distributing space among items and aligning them within a container along a single row or column, replacing older alignment hacks like floats and `display: table`.

## Key Concepts

- `display: flex` on a parent turns its direct children into flex items laid out along a single axis (row or column).
- `flex-direction` sets the main axis (`row`/`row-reverse`/`column`/`column-reverse`).
- `justify-content` aligns items along the main axis; `align-items` aligns items along the cross axis.
- `flex-wrap` controls whether items wrap onto new lines when they don't fit on one.
- Individual items control their own sizing via `flex-grow`, `flex-shrink`, and `flex-basis` (commonly set together via the `flex` shorthand).

## Interview Questions & Answers

### Q1. What is Flexbox and how does it help with alignment?

#### Answer

Flexbox (Flexible Box Layout) is a one-dimensional layout model — it lays items out along a single axis at a time, either a row or a column — designed for distributing space among items and aligning them within a container.

Applying `display: flex` to a parent turns its direct children into flex items. The container then controls the overall layout:

- `flex-direction` sets the main axis (`row` or `column`).
- `justify-content` aligns items along the main axis (e.g. `center`, `space-between`, `space-around`).
- `align-items` aligns items along the cross axis (e.g. `center`, `stretch`, `flex-start`).
- `flex-wrap` controls whether items wrap onto new lines instead of overflowing or shrinking indefinitely.

Individual items can further control their own sizing with `flex-grow` (how much they expand into available space), `flex-shrink` (how much they shrink when space is tight), and `flex-basis` (their starting size) — usually set together via the `flex` shorthand.

Flexbox solved long-standing alignment pain points that used to require hacks like floats plus clearfixes, `display: table`, or manual absolute-positioning math — most notably vertical centering, equal-height columns, and evenly distributing leftover space between items.

#### Code Example

```css
.container {
  display: flex;
  justify-content: center; /* center along main axis */
  align-items: center; /* center along cross axis */
  height: 100vh;
}
```

```html
<div class="container">
  <div>I'm centered both horizontally and vertically</div>
</div>
```

#### Follow-up Questions

- How does Flexbox differ from CSS Grid, and when would you reach for one over the other?
- What's the difference between `align-items` and `align-content` when items wrap?
- How do `flex-grow`, `flex-shrink`, and `flex-basis` interact when the `flex` shorthand is used (e.g. `flex: 1`)?

## Common Pitfalls

- Reaching for Flexbox for two-dimensional layouts (rows and columns simultaneously) when CSS Grid is the better fit, since Flexbox is fundamentally one-dimensional.
- Forgetting `flex-wrap: wrap` and being surprised items shrink indefinitely or overflow instead of moving to a new line.
- Confusing `justify-content` (main axis) with `align-items` (cross axis) — swapping them has no effect until `flex-direction` changes.
- Not accounting for `min-width`/`min-height: auto` defaults on flex items, which can prevent content from shrinking as expected inside a flex container.
