# Microfrontend

> Part of: 25-Architecture

## Overview

Micro-frontends apply the microservices idea to the frontend: splitting a
large web application into independently developed, tested, and deployed
pieces, owned by separate teams, that are composed together at runtime (or
build time) into a single product the user experiences as one app. The most
common runtime composition mechanism today is
[Module Federation](04-Module-Federation.md).

## Key Concepts

- **Independent deployability** is the defining property — each team can
  ship its slice on its own schedule without coordinating a release with
  every other team, unlike a single monolithic frontend release.
- **Composition approaches**: runtime (Module Federation, iframes, web
  components), build-time (installing another team's app as an npm
  package), and server-side (edge/SSR composition of fragments).
- Ownership boundaries are usually drawn along business domains or routes
  (e.g. "checkout team owns `/checkout/*`"), not technical layers.
- Micro-frontends solve an *organizational* scaling problem (many teams,
  one product) more than a technical one — introducing them for a small,
  single-team app usually adds cost without benefit.

## Interview Questions & Answers

### Q1. Why use micro-frontends?

#### Answer

Micro-frontends are worth the added complexity mainly when multiple teams
need to ship one product without blocking on each other. Benefits:

- **Independent deployability** — each team releases on its own cadence.
- **Team autonomy / ownership** — a team owns a vertical slice end-to-end
  (UI, tests, deploy), which scales better organizationally than everyone
  contributing to one shared codebase.
- **Tech-stack independence** — teams can upgrade a framework version, or in
  principle use a different framework, in their own slice without a
  coordinated big-bang migration.
- **Incremental legacy migration** — the "strangler fig" pattern: carve
  a new remote out of a legacy monolith one route/feature at a time,
  instead of a risky full rewrite.
- **Smaller, faster CI** per team, since each pipeline only builds/tests its
  own slice.

Trade-offs to weigh against this: risk of duplicated dependencies inflating
bundle size, more infrastructure/tooling complexity, and harder cross-app
consistency (design, versioning, testing). It's generally only worth it once
an organization has enough teams that a single shared frontend repo becomes
the bottleneck.

### Q2. Module Federation vs iframes vs Web Components — how do these compare as micro-frontend composition mechanisms, and how do you manage shared dependencies without version conflicts?

#### Answer

**iframes** give the strongest isolation — separate document, JS realm,
and CSS, so nothing leaks either direction — at the cost of the weakest
integration: no shared DOM (a dropdown or tooltip can't render outside the
iframe's box), no code sharing (every iframe ships its own full bundle,
including its own framework runtime), and awkward routing/history,
resizing, and SEO. Cross-frame communication is limited to `postMessage`.
Best fit: embedding something genuinely untrusted or fully independent
(e.g. a third-party payment form), where isolation matters more than
integration.

**Web Components** (Custom Elements + Shadow DOM) are framework-agnostic:
a remote ships a `<checkout-widget>` custom element any host — React,
Angular, plain HTML — can mount, with Shadow DOM giving CSS encapsulation
without a full iframe's isolation cost. The trade-off: there's no built-in
mechanism to share a framework runtime, so each Web Component often bundles
its own React/Angular instance unless deliberately engineered not to
(inflating total bundle size), and cross-component communication is
limited to DOM events/attributes rather than a rich JS API.

**Module Federation** is the most integrated option: host and remotes share
the same JS realm and can share a singleton instance of a framework/library
(`shared: { react: { singleton: true } }` — see
[Module Federation](04-Module-Federation.md)), so remotes can access the
host's context/state directly and avoid duplicate framework copies. The
trade-off is the least isolation — a bug in one remote runs in the same
realm as everything else — and it works best within one framework
ecosystem, unlike Web Components' framework-agnosticism.

**Choosing between them:** Module Federation when every micro-frontend
shares one framework and needs tight integration (shared design system,
shared auth context, shared routing) — the common case for most
enterprise micro-frontend setups. Web Components when composition must be
framework-agnostic (teams on different stacks, or embedding in a CMS/
marketing site). iframes when isolation/security matters more than
integration (untrusted third-party content).

**Shared dependency management and version conflicts** are really a
Module-Federation-specific concern, since it's the only one of the three
that shares a JS realm at all — see
[Module Federation Q2/Q3](04-Module-Federation.md) for the `singleton`/
`requiredVersion`/`strictVersion` mechanics and how to catch conflicts in
CI before they reach production. Deployment independence without version
conflicts additionally relies on deploying each remote to an immutable,
versioned URL (covered in Q6/Q7 below) so a remote's independent release
can't silently break the host or another remote through an incompatible
shared-library version.

#### Follow-up Questions

- Why can't a Web Component easily share a single React instance across
  multiple components the way Module Federation can?
- If a team insists on using a different framework for their remote, which
  of these three options remains viable, and what do they give up?

### Q3. Draw/describe a typical micro-frontend architecture.

#### Answer

A common shape is a **shell (container) app** that owns the page shell,
top-level routing, and shared cross-cutting concerns (auth, shared
dependencies), and dynamically loads independently deployed **remote apps**
into designated regions of the page:

```
                        ┌─────────────────────────────┐
                        │        Shell / Container      │
                        │  (routing, auth, layout,       │
                        │   shared deps: React, design   │
                        │   system, event bus)           │
                        └───────────────┬─────────────┘
                                        │ loads remoteEntry.js at runtime
              ┌─────────────────────────┼─────────────────────────┐
              ▼                         ▼                         ▼
      ┌───────────────┐         ┌───────────────┐         ┌───────────────┐
      │  Remote: Search │         │ Remote: Cart  │         │ Remote: Checkout│
      │  (own repo/CI)  │         │ (own repo/CI) │         │  (own repo/CI)  │
      └───────────────┘         └───────────────┘         └───────────────┘
```

Each remote is built and deployed independently (its own repo, CI pipeline,
and release cadence), and publishes a manifest (e.g. `remoteEntry.js`) that
the shell fetches and loads at runtime — so shipping a new version of the
Cart remote doesn't require rebuilding or redeploying the shell.

### Q4. How do micro-frontends communicate with each other?

#### Answer

Since remotes are meant to be loosely coupled (ideally deployable and
testable in isolation), direct imports between two remotes are avoided.
Common communication mechanisms instead:

- **Custom events / event bus** — `window.dispatchEvent(new CustomEvent(...))`
  and listeners, or a small shared pub-sub module, for one-way notifications
  ("cart updated") without either side knowing about the other's internals.
- **Shared state via the shell** — the shell owns a store (or a slice of
  one) and passes relevant state/callbacks down as props or context to each
  mounted remote.
- **Browser storage** — `localStorage`/`sessionStorage` plus the `storage`
  event for cross-tab/cross-app state that should persist.
- **URL/query params** — encoding state in the URL when it should be
  shareable/bookmarkable and naturally scoped to routing.
- **A shared library exposed via Module Federation** — e.g. an `auth` or
  `analytics` module every remote consumes the same instance of, instead of
  each reimplementing it.

The general principle: prefer the least coupling that gets the job done, and
avoid a remote reaching into another remote's internals directly.

### Q5. How is routing handled across micro-frontends?

#### Answer

Two common patterns:

- **Shell-owned routing** — the shell holds the single router instance and
  browser history; it matches the URL to a remote and mounts/lazy-loads that
  remote for the current route. This avoids the classic problem of two
  router instances fighting over `history` (double navigation, back-button
  breaking).
- **Nested routing within a remote** — once the shell mounts a remote for
  its base path (e.g. everything under `/checkout/*`), that remote can own
  its own nested routes beneath that path without the shell needing to know
  about them.

Only one router should control the actual browser `history` object at a
time; a remote's internal router typically runs in a mode where it defers to
(or is scoped under) the shell's history rather than instantiating its own
competing listener.

### Q6. How is authentication handled across micro-frontends?

#### Answer

Auth is usually centralized in the shell rather than reimplemented per
remote:

- The shell owns the login flow and the session/token, and exposes identity
  (user, permissions, a fetch-with-auth wrapper) to remotes via shared
  context/props, or via a shared auth module published through Module
  Federation — so no remote implements its own login screen or token
  refresh logic.
- Token storage favors an httpOnly cookie over `localStorage` where
  possible, since a cookie is automatically sent with requests and isn't
  reachable by remote JS (reducing XSS blast radius) — relevant because a
  compromised or buggy remote is one more surface that could read a token
  stored in JS-accessible storage.
- If remotes are hosted on different origins/subdomains, SSO (shared cookie
  domain, or a token-exchange step) keeps the user from re-authenticating
  per remote.

### Q7. What deployment strategies work for micro-frontends?

#### Answer

Each remote is deployed through its own independent CI/CD pipeline, decoupled
from the shell and from other remotes:

- The shell fetches each remote's manifest (`remoteEntry.js`) at **runtime**,
  so a new remote version goes live for users without a shell rebuild or
  redeploy.
- Remote assets are typically hosted on a CDN, with cache-busting (a
  content-hashed or versioned filename/path) so the shell always resolves to
  the intended build, and old builds stay available at their own URLs.
- Rollouts can be gradual — feature-flagging or canarying which manifest
  version a percentage of users/shells resolve to, before flipping
  everyone over.

### Q8. What's the rollback strategy for micro-frontends?

#### Answer

Because a remote is resolved by URL/manifest at runtime rather than baked
into the host's build, rollback doesn't require a host redeploy either:

- Deploy each remote version to an **immutable, versioned path** (not
  overwritten in place), and keep the previous N versions live on the CDN.
- Rollback = re-point the manifest/config the shell reads (or the version a
  feature flag serves) back to the last known-good version — effectively an
  instant "kill switch" rather than a rebuild-and-redeploy cycle.
- Combine with canary/percentage rollout so a bad remote version only
  affects a small slice of traffic before automated or manual rollback
  kicks in, rather than every user hitting it at once.

## Common Pitfalls

- Introducing micro-frontends for a small, single-team app, where the
  added infra/tooling complexity outweighs any benefit.
- Letting two remotes import from each other directly, recreating tight
  coupling under a micro-frontend label.
- Running two independent router instances that both try to own browser
  history, causing back-button and navigation bugs.
- Each remote implementing its own auth/login instead of centralizing it in
  the shell, leading to inconsistent session handling and duplicated logic.
- Deploying a remote by overwriting its existing URL in place, which makes
  rollback impossible without a new deploy.

## References

- [Micro Frontends (martinfowler.com)](https://martinfowler.com/articles/micro-frontends.html)
- [micro-frontends.org](https://micro-frontends.org/)
