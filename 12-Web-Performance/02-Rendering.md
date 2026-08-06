# Rendering

> Part of: 12-Web-Performance

## Overview

Rendering performance is about keeping the DOM small and updates cheap,
which matters most for large lists/tables where naively rendering every
row tanks scroll and interaction performance regardless of how fast the
data fetch itself is.

## Key Concepts

- **Virtualization/windowing** — render only the rows currently in (or
  near) the viewport, recycling a fixed pool of DOM nodes as the user
  scrolls, so DOM size stays constant regardless of dataset size.
- **Fixed vs variable row height** — fixed height makes scroll-position
  math trivial (`index * height`); variable height needs a measurement
  pass (cached per-row, e.g. via `ResizeObserver`) and risks layout jumps
  if size estimates are wrong.
- **Pagination vs infinite scroll** — pagination gives a bounded DOM,
  deep-linkable pages, and simpler back-button/scroll-restoration behavior;
  infinite scroll feels more fluid but still needs virtualization
  underneath once enough pages have loaded, plus extra work for scroll
  restoration and accessibility.
- **Off-main-thread filtering/sorting** — debouncing input, moving heavy
  computation server-side or into a Web Worker, and chunking any
  unavoidable main-thread work so it yields between chunks.

## Interview Questions & Answers

### Q1. How do you optimize a page with a large data table (10k+ rows)?

#### Answer

**Never render all 10k rows to the DOM.** Even if the data fetch is fast,
DOM node count itself is what kills scroll and interaction performance.
Use virtualization/windowing (`react-window`, TanStack Virtual, or
Angular CDK's virtual scroll): only the rows currently in the viewport
plus a small overscan buffer are actually mounted, and DOM nodes are
recycled as the user scrolls — so the DOM stays at roughly the same size
(~30-50 rows) no matter how large the dataset is.

```js
import { FixedSizeList as List } from 'react-window';

function Table({ rows }) {
  const Row = ({ index, style }) => (
    <div style={style}>{rows[index].name}</div>
  );

  return (
    <List height={600} itemCount={rows.length} itemSize={35} width="100%">
      {Row}
    </List>
  );
}
```

**Fixed vs variable row height.** Fixed height is simplest — scroll
position is just `index * rowHeight` math. Variable row height (e.g. rows
with wrapping text) needs a measurement pass, caching each row's measured
height so scroll math stays correct, and risks a visible jump if an
initial size estimate is wrong before measurement completes.

**Pagination vs infinite scroll.** Pagination gives a bounded DOM,
easy deep-linking (page number in the URL), and simple back-button
behavior. Infinite scroll feels more fluid for a "scan everything" flow
but complicates scroll-position restoration on back-navigation,
accessibility (keyboard/screen-reader navigation to a specific row), and
still needs virtualization once enough pages have loaded to avoid
unbounded DOM growth. For a data table specifically — as opposed to a
social feed — pagination (or virtualized infinite scroll) is usually the
more predictable and accessible choice.

**Keeping filter/sort off the main thread:**

- Debounce/throttle the filter input so every keystroke doesn't trigger a
  full re-computation.
- Push filtering/sorting **server-side** once the dataset is large enough
  that client-side compute becomes the bottleneck — request just the
  filtered/sorted/paginated slice instead of shipping all 10k rows to the
  client in the first place.
- If filtering/sorting must stay client-side (data's already fully
  loaded), move the actual computation into a **Web Worker** so
  typing/scrolling stays responsive while the sort/filter runs, posting the
  result back when done.
- For any unavoidable main-thread work, chunk it with
  `requestIdleCallback`/scheduler-based batching so it yields between
  chunks instead of blocking a long frame.

Also worth doing: memoizing row renderers keyed by row id (`React.memo`)
so an update to one row's data doesn't re-render every visible row, and
precomputing derived/formatted values once when data arrives instead of
inline in the render path.

#### Follow-up Questions

- How would your virtualization approach change if rows have variable,
  unknown-until-render height (e.g. wrapping text)?
- At what row count would you push filtering/sorting server-side instead
  of doing it client-side?
- How would you handle a "select all 10k rows" action without loading all
  10k rows into the DOM to do it?

## Common Pitfalls

- Rendering all rows to the DOM and relying on the browser to "just handle
  it," which degrades scroll/interaction performance well before 10k rows.
- Virtualizing rows but still running full-dataset filter/sort on every
  keystroke on the main thread, blocking input while it computes.
- Infinite scroll without virtualization underneath, so the DOM keeps
  growing as more pages load until performance degrades anyway.
- Re-rendering every visible row on any data change instead of memoizing
  per-row by a stable key.

## References

- [TanStack Virtual docs](https://tanstack.com/virtual/latest)
- [web.dev: Virtualize large lists](https://web.dev/articles/virtualize-long-lists-react-window)
