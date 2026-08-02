# 04-CSS — FAQs

> Quick-fire Q&A for 04-CSS. Keep answers short (2-5 lines); link to the relevant topic file for depth.

## Q1. What is CSS specificity, and how does cascading/priority work?

**A:** Specificity determines which CSS rule "wins" when multiple rules target the same element with conflicting declarations. It's calculated as a tuple weighting selector types from highest to lowest: inline styles > ID selectors (`#id`) > class/attribute/pseudo-class selectors (`.class`, `[attr]`, `:hover`) > element/pseudo-element selectors (`div`, `::before`). Higher-specificity selectors always win regardless of source order; among equal-specificity rules, the one declared later wins — and `!important` overrides normal specificity entirely, so it should be used sparingly.

## Q2. What is the difference between `em` and `rem` units?

**A:** `em` is relative to the font-size of its own element's parent — nested `em` values compound, so a `2em` inside a `2em` inside a `16px` base becomes `64px`, which can surprise you in deeply nested components. `rem` ("root em") is always relative to the root `<html>` element's font-size, regardless of nesting, making it more predictable for consistent sizing across a component tree. This is why `rem` is generally preferred for font-sizing/spacing in modern CSS, while `em` is still useful for sizing that should scale relative to its own component's local font-size (e.g. icon sizing relative to adjacent text).
