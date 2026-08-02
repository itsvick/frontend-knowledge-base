# Architecture

> Subsection of 05-React.

## Overview

How a React codebase is organized as it grows — folder structure and where
state should live — so the project stays navigable as features and team
size increase.

## Key Concepts

- There's no single "correct" structure; the right one depends on app
  size/team, but consistency matters more than the specific convention
  chosen.
- **Feature-based** (a.k.a. "package by feature") groups files by domain
  feature (`features/checkout/`) rather than by file type (`components/`,
  `reducers/`); it tends to scale better than type-based structuring for
  medium-to-large apps.
- **State colocation** — keep state as close as possible to where it's
  used; only lift it up (or into global state) once multiple distant
  components actually need it.

## Interview Questions & Answers

### Q1. Which project folder-structuring practice is considered best for React projects?

#### Answer

There isn't one universally "correct" structure, but two common conventions
and their trade-offs:

- **Type-based** (`components/`, `hooks/`, `reducers/`, `services/`) —
  groups files by what they *are*. Simple for small apps, but as the app
  grows, working on one feature means jumping across many top-level
  folders.
- **Feature-based / domain-based**
  (`features/checkout/{components,hooks,api}`,
  `features/profile/...`) — groups files by what they *do*, colocating a
  feature's components, hooks, and API calls together. Scales better for
  medium-to-large apps since a feature can be understood, tested, or
  deleted as one unit.

Most teams building anything beyond a small app converge on feature-based
structure, often with a small set of shared/top-level folders (`shared/`,
`lib/`, `app/`) for cross-feature code (design-system components,
utilities, routing setup).

#### Follow-up Questions

- How do you decide when a piece of state belongs in a feature folder vs.
  global state?

## Common Pitfalls

- Over-engineering folder structure for a small app before it needs the
  extra layering.
- Mixing type-based and feature-based structuring inconsistently across the
  same codebase.
