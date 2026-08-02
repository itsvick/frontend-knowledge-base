# Design System

> Part of: 25-Architecture

## Overview

A design system is a shared source of truth — design tokens, a component library built on those tokens, usage documentation, and a governance process — that keeps a product visually and behaviorally consistent across teams. In an AI-driven development workflow, it doubles as the contract that coding agents read and follow when generating UI code.

## Key Concepts

- **Design tokens** — colors, spacing, typography, radii, shadows, etc. expressed as named variables (not hardcoded values), shared between design tools (e.g. Figma variables) and code (e.g. CSS custom properties, a JS/TS token file).
- **Component library** — built on top of tokens, with variants/states (size, style, disabled, loading, etc.) defined once per component and reused everywhere, instead of one-off styling per usage site.
- **Usage documentation** — guidelines on when/how to use each component (do's/don'ts), not just a props/API reference; misuse (right component, wrong context) is the most common design-system failure mode.
- **Governance/versioning** — a clear process for proposing new components or shipping breaking changes, so consuming teams can trust the system won't shift unpredictably underneath them.
- **AI legibility** — in an AI-assisted workflow, tokens/components need clear, descriptive, discoverable names and machine-readable usage patterns (Storybook stories, Figma Code Connect) so coding agents can find and reuse them correctly instead of inventing new patterns.
- **Automated guardrails** — lint rules banning raw color/spacing values, visual regression testing, and accessibility linting matter more with AI-generated code, since a human reviewer can't manually verify every value in a much higher volume of generated changes.

## Interview Questions & Answers

### Q1. What makes a good design system, especially in an AI-driven development environment?

#### Answer

At its core, a good design system rests on four pillars, regardless of how the code is produced:

- **Design tokens** — colors, spacing, typography, radii, shadows, etc. expressed as named variables rather than hardcoded values, acting as the single source of truth shared between design tools (Figma) and code. A button's blue isn't `#1a73e8` sprinkled across files; it's `color.action.primary`, defined once and consumed everywhere.
- **A component library built on those tokens** — with clear variant/state coverage defined once and reused everywhere. A `Button`'s size, variant, disabled, and loading states are all specified in one place, rather than each screen re-implementing its own slightly-different button.
- **Documentation that shows when/how to use each component** — usage guidelines and do's/don'ts, not just an API reference. Misuse — picking the wrong component for the context (e.g. a destructive action styled as a primary button) — is the most common way design systems fail in practice, more common than missing props or styling bugs.
- **Governance and versioning** — a clear, predictable process for proposing new components or making breaking changes. A design system that shifts unpredictably underneath consuming teams quickly loses their trust and gets forked or bypassed.

What changes in an AI-driven environment is less the pillars themselves and more how strictly they need to be enforced and how legible they are to a model, not just to humans:

- AI coding agents (Copilot, Claude Code, Cursor) generate UI fastest and most consistently when the design system is *legible to the model* — clear, descriptive, discoverable component and token names, plus machine-readable usage patterns (Storybook stories, or Figma Code Connect mapping design components directly to code snippets) that an agent can read and follow, rather than tribal knowledge locked in a designer's head or an undocumented Figma file.
- A strict, well-enforced token system reduces the "AI invents a slightly-off hardcoded value" failure mode. If the agent is nudged — via lint rules or an explicit instruction/config file — to always reach for a token instead of a raw hex or pixel value, generated code stays visually consistent even across many different prompts and sessions.
- Automated checks matter more, not less, with AI-generated code: lint rules banning raw color/spacing values outside the token file, visual regression testing, and accessibility linting. A human is now reviewing a much higher volume of generated UI changes and can't manually verify every value by eye, so the design system's guardrails have to catch what review misses.

In short: the fundamentals (tokens, components, docs, governance) don't change, but AI-assisted development raises the cost of ambiguity and the cost of unenforced rules, since both are now amplified across many more generated changes than a human team alone would produce.

#### Follow-up Questions

- How would you set up Figma Code Connect (or an equivalent) so an AI agent can reliably map a design to the correct component in code?
- What lint rules would you add specifically to stop AI-generated code from hardcoding values instead of using tokens?
- How do you handle a case where an AI agent invents a new component variant that doesn't exist in the design system?

## Common Pitfalls

- Hardcoding colors, spacing, or font sizes instead of referencing tokens — breaks the single source of truth and is the exact failure mode AI-generated code is especially prone to without lint enforcement.
- Treating documentation as an API reference only, without usage guidelines — leads to components being used in the wrong context, the most common real-world design-system failure.
- Skipping governance/versioning, so consuming teams get surprised by breaking changes and start bypassing the system entirely.
- Assuming an AI coding agent will "just know" the design system — without legible naming, Storybook stories, or Code Connect mappings, agents fall back to inventing ad hoc markup and styles instead of reusing existing components.
- Not adding automated guardrails (lint rules, visual regression, a11y checks) to match the increased volume of AI-generated UI changes, relying instead on manual review to catch inconsistencies.
