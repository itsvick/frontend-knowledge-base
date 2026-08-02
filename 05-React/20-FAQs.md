# 05-React — FAQs

> Quick-fire Q&A for 05-React. Keep answers short (2-5 lines); link to the relevant topic file for depth.

## Q1. Which browser tool is essential for debugging and profiling React components?

**A:** React Developer Tools (the official Chrome/Firefox extension) — its
Components tab inspects the fiber tree, props/state/hooks, and its Profiler
tab records renders to show what re-rendered and why. See
[[13-Performance]] Q11 for using it to debug unnecessary renders.

## Q2. What's a key benefit of using a UI component library like MUI or Ant Design?

**A:** Pre-built, accessible, tested components (buttons, dialogs, tables,
forms) with a consistent design language out of the box, cutting the time
spent building and maintaining low-level UI primitives so teams can focus on
app-specific logic. Trade-offs: added bundle size and less control over
exact markup/styling versus hand-rolled components.
