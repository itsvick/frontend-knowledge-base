# Module Federation

> Part of: 25-Architecture

## Overview

Module Federation is a webpack 5 (also supported by Rspack and Vite via
plugins) capability that lets multiple independently-built and
independently-deployed JavaScript applications share code **at runtime**,
without publishing/installing packages or rebuilding the consuming app. It's
the most common technical mechanism used to implement
[micro-frontends](03-Microfrontend.md).

## Key Concepts

- **Host** vs **remote** — a host app consumes modules exposed by one or more
  remote apps at runtime, typically by fetching a `remoteEntry.js` manifest
  the remote publishes.
- **`exposes`** — config on the remote declaring which modules/components it
  makes available to consumers.
- **`remotes`** — config on the host declaring which remote apps (and which
  exposed modules) it wants to consume.
- **`shared`** — dependencies (e.g. `react`, `react-dom`) that can be shared
  across host and remotes instead of each bundling its own copy, avoiding
  duplicate downloads and duplicate library instances.
- Because remotes are resolved at runtime via a URL/manifest, a remote can be
  redeployed independently without requiring a host rebuild.

## Interview Questions & Answers

### Q1. What is Module Federation and what problem does it solve?

#### Answer

Module Federation lets separately built and deployed apps share JavaScript
modules with each other at runtime instead of at build time. Without it,
sharing code across independently deployed frontends means publishing to an
npm registry and re-installing/rebuilding on every change, which couples
everyone's deploy cadence together. Module Federation solves this by letting
a **remote** app expose modules (e.g. a component, a page, a utility) that a
**host** app loads dynamically over the network at runtime — so the remote
can ship a new version and the host picks it up on next load, with no
host rebuild required.

#### Code Example

```js
// remote app's webpack.config.js
new ModuleFederationPlugin({
  name: "productsApp",
  filename: "remoteEntry.js",
  exposes: {
    "./ProductList": "./src/ProductList",
  },
  shared: { react: { singleton: true }, "react-dom": { singleton: true } },
});

// host app's webpack.config.js
new ModuleFederationPlugin({
  name: "shell",
  remotes: {
    productsApp: "productsApp@https://cdn.example.com/products/remoteEntry.js",
  },
  shared: { react: { singleton: true }, "react-dom": { singleton: true } },
});

// host usage
const ProductList = React.lazy(() => import("productsApp/ProductList"));
```

### Q2. How does Module Federation handle shared dependencies?

#### Answer

The `shared` config lets host and remotes avoid bundling duplicate copies of
the same library. At runtime, before loading a remote, webpack checks
whether a compatible version of a shared dependency is already loaded in the
share scope; if so, it reuses that instance instead of loading the remote's
own copy. Key options:

- **`singleton: true`** — force exactly one instance across host and all
  remotes, even if requested versions differ slightly. This is required for
  libraries that break when duplicated, like React (duplicate React copies
  break hooks/context because they no longer share the same module
  internals).
- **`requiredVersion`** — the semver range this app expects; used to decide
  whether an already-loaded version satisfies the requirement.
- **`eager: true`** — bundle the dependency into the initial chunk instead of
  loading it asynchronously (needed for the host's own entry, since the
  entry can't itself be async).

#### Follow-up Questions

- What breaks if two remotes load different major versions of a non-singleton
  shared library?

### Q3. How do you handle version conflicts between the host and remotes?

#### Answer

Version conflicts happen when the host and a remote (or two remotes) expect
incompatible versions of the same shared dependency. Common strategies:

- For libraries that can't tolerate multiple instances (React, a design
  system, a global state store), mark them `singleton: true` with a
  `requiredVersion` range, and use `strictVersion: true` in production so an
  incompatible version fails loudly at load time instead of silently causing
  subtle bugs (duplicate context, broken hooks).
- For libraries that tolerate multiple instances (utility libs, isolated
  widgets), leave them non-singleton — each remote can load its own
  compatible version without affecting others.
- Keep a small "shared contract" (which libraries are shared and at what
  version range) documented and versioned in lockstep across teams, since
  it's effectively a cross-team API.
- Catch conflicts early with automated checks in CI (e.g. a script that
  diffs each remote's shared-dependency versions against the host's allowed
  ranges) rather than discovering them at runtime in production.

## Common Pitfalls

- Not marking React (or another context-sensitive library) as `singleton`,
  causing duplicate instances and broken hooks/context that only surface at
  runtime.
- Treating `remoteEntry.js` as immutable/cacheable by URL without a
  versioned path or cache-busting, so users can get a stale or
  partially-updated remote.
- Forgetting that shared-version mismatches without `strictVersion` fail
  silently (webpack just loads a second copy) instead of erroring, making the
  bug hard to trace back to Module Federation.

## References

- [webpack Module Federation docs](https://webpack.js.org/concepts/module-federation/)
