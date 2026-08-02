# Monorepo

> Part of: 26-Architecture

## Overview

A **monorepo** is a single repository that holds multiple distinct projects/packages (apps, libraries, tools) instead of splitting each into its own repo (**polyrepo**). Packages live side by side under one version-control history, share tooling/config, and can reference each other directly through a package manager's **workspaces** feature. On top of workspaces, build orchestrators like **Nx** and **Turborepo** add a dependency-graph-aware task runner, so builds/tests/lints run only for what actually changed, in the correct order, with results cached and reused.

Compared to a polyrepo, a monorepo makes cross-package changes atomic (one PR can update a shared library and every app that consumes it), simplifies dependency version alignment, and enables code sharing without publishing internal packages to a registry — at the cost of needing tooling that scales task execution and caching as the repo grows.

## Key Concepts

- Workspaces (npm/Yarn/pnpm) let multiple packages in one repo be installed together and reference each other by name via symlinks, without publishing to a registry
- Build orchestrators (Nx, Turborepo, Lerna) sit on top of workspaces to add dependency-graph-aware, cached task running
- The **dependency graph** models which package depends on which, driving task ordering and "affected" detection
- **Caching** hashes a task's inputs and reuses prior output on a hit — locally, and across machines/CI with remote/distributed caching
- **Incremental builds** rebuild/retest only what changed (and what depends on it), not the whole repo
- **Shared libraries** are internal packages (UI kit, utils, types) consumed by multiple apps via the workspace link instead of duplicated code
- Publishing selectively pushes specific packages from a (often private) monorepo out to a registry like npm, rewriting internal `workspace:*` ranges to real semver first

## Interview Questions & Answers

### Q1. What are Yarn workspaces and how do they work?

#### Answer

Yarn workspaces is a built-in feature (Yarn Classic and Berry) for managing multiple packages in one repo with a single install. The root `package.json` declares a `"workspaces"` array of glob patterns pointing at package folders. Running `yarn install` once:

- Installs all packages' dependencies together and **hoists** shared dependencies up to the root `node_modules` to dedupe them.
- **Symlinks** each internal package into the others' `node_modules`, so `packages/app` can `import` `packages/ui` by its package name as if it were an installed dependency, with no publish step.

#### Code Example

```json
// root package.json
{
  "private": true,
  "workspaces": ["packages/*"]
}
```

```json
// packages/app/package.json
{
  "name": "app",
  "dependencies": { "ui": "*" }
}
```

#### Follow-up Questions

- How does Yarn's hoisting cause "phantom dependency" bugs (a package using a dependency it never declared, because hoisting happened to place it in reach)?
- What changes with Yarn Berry's Plug'n'Play mode, and why was it introduced?

### Q2. What are pnpm workspaces, and how do they differ from npm/Yarn workspaces?

#### Answer

pnpm workspaces are configured via a `pnpm-workspace.yaml` file at the repo root listing package globs. Internal packages reference each other with the `workspace:*` protocol in `package.json`.

The key difference is how dependencies are laid out on disk. npm/Yarn hoist dependencies into a largely flat root `node_modules`, which lets a package accidentally `require()` a dependency it never declared (a **phantom dependency**) just because hoisting placed it within reach. pnpm instead keeps a global **content-addressable store** (one copy of each package version on disk, hard-linked wherever it's needed) and builds a strict, symlinked `node_modules` per package that only exposes the dependencies that package actually declares — phantom access fails immediately. This also makes installs faster and far more disk-efficient across projects that share dependency versions.

#### Code Example

```yaml
# pnpm-workspace.yaml
packages:
  - "packages/*"
```

```json
// packages/app/package.json
{
  "dependencies": { "ui": "workspace:*" }
}
```

#### Follow-up Questions

- Why can't a `workspace:*` version range be published as-is to a public registry?
- Why is pnpm's strict `node_modules` considered safer than npm/Yarn's hoisting model?

### Q3. Nx vs Turborepo — how do they compare?

#### Answer

Both are **monorepo build systems** that sit on top of a package manager's workspaces (npm/Yarn/pnpm) to add task orchestration, caching, and dependency-graph awareness. Neither replaces workspaces — they consume the graph workspaces already define.

**Turborepo** is lightweight and framework-agnostic: a `turbo.json` defines a pipeline of tasks and their ordering (e.g. `build` depends on `^build`, meaning a package's dependencies build first), plus local and remote caching. Minimal config, small learning curve, good fit for a handful of apps/packages.

**Nx** is a more full-featured platform: code generators/scaffolding, framework-specific plugins (Angular, React, Next), a visual dependency-graph explorer (`nx graph`), and enforceable **module boundary** lint rules to keep architecture consistent across teams. It also does computation caching (locally and via Nx Cloud). It suits larger, multi-team orgs that want generated conventions and enforced structure, not just faster builds.

Rule of thumb: reach for **Turborepo** for speed/simplicity with a small-to-medium set of packages; reach for **Nx** when you need generators, enforced boundaries, and richer graph tooling at scale.

#### Follow-up Questions

- How does each tool compute the set of "affected" projects for a given change?
- What would make you migrate from Turborepo to Nx (or vice versa) on an existing repo?

### Q4. What is a dependency graph in a monorepo, and why does it matter?

#### Answer

The dependency graph is the explicit map of which package/project depends on which others, derived from `package.json` dependencies (and, in Nx, also static analysis of import statements). Build orchestrators use it to:

- **Order tasks correctly** — a package's `build` only runs after every package it depends on has finished building.
- **Compute "affected" projects** — given a git diff against a base branch, walk the graph outward from directly-changed packages to every package that (transitively) depends on them, so CI only builds/tests what could actually be impacted.
- **Enforce architecture** — module-boundary rules (e.g. Nx) can block a "feature" library from importing another "feature" library directly, using the same graph.

Both Nx (`nx graph`) and Turborepo (`turbo run build --graph`) can render this graph visually for debugging.

#### Follow-up Questions

- How would you detect and break a circular dependency between two packages in the graph?
- What happens to build correctness if a real dependency is missing from the graph (e.g. a runtime-only dependency not declared in `package.json`)?

### Q5. What are incremental builds, and how do monorepo tools implement them?

#### Answer

An incremental build rebuilds/retests only the packages affected by a change, instead of the entire repo. It combines three layers:

1. **Affected detection** — compute the changed packages from a git diff, then expand to their dependents via the dependency graph.
2. **Task-level caching** — even within an affected run, a task whose inputs are unchanged from a prior run is restored from cache instead of re-executed.
3. **Compiler/tool-level incrementality** — e.g. TypeScript's `--incremental`/composite project references cache a `.tsbuildinfo` file so unchanged files skip re-type-checking, independent of the monorepo tool.

#### Code Example

```bash
# Turborepo: only run build for packages changed since origin/main
turbo run build --filter=...[origin/main]

# Nx: only run build for projects affected by the current changes
nx affected -t build
```

#### Follow-up Questions

- How do you keep the "affected" calculation correct when a shared config file (e.g. root `tsconfig.json`, ESLint config) changes and isn't tied to a specific package?

### Q6. How does caching work in tools like Nx and Turborepo?

#### Answer

Each task run (e.g. `build` for package `ui`) is hashed from its inputs — source files, the outputs of its dependencies, relevant environment variables, and the task's own config/lockfile entry. If an entry with that exact hash already exists in the cache, the tool restores the previous output (build artifacts and stdout) instead of re-running the task — this works even if that exact task ran on a teammate's machine or in a different CI job, as long as a **remote/distributed cache** (Nx Cloud, Turborepo Remote Cache) is configured, so the work is done once and shared everywhere. Without a remote cache, caching is still useful locally (disk cache under `.turbo` or `node_modules/.cache`), just not shared across machines.

#### Follow-up Questions

- What must be included in a task's cache key for the cache to stay correct (not just fast)? What happens if an env var that affects the build output is left out?
- How do you invalidate a bad or stale cache entry safely?

### Q7. What are common build optimization strategies in a large monorepo?

#### Answer

- Run **affected-only** builds/tests in CI instead of the whole repo on every change.
- Use **remote/distributed caching** so CI runners and developer machines share task outputs instead of recomputing them.
- **Parallelize** independent tasks across the dependency graph (anything not blocking on another task's output can run concurrently).
- Split CI into **per-project pipelines or matrix jobs** driven by the affected graph, rather than one monolithic job.
- Use faster per-package compilers/bundlers (esbuild, SWC) instead of slower ones (Babel, tsc-only) where output correctness allows.
- Use TypeScript **project references**/composite builds so unaffected packages skip re-type-checking.
- **Prune** the workspace for deployment artifacts (e.g. `turbo prune`) so a Docker image only bundles the subset of the monorepo a given service actually needs, instead of the whole repo.

### Q8. What are shared libraries in a monorepo, and how should they be structured?

#### Answer

Shared libraries are internal packages (e.g. `packages/ui`, `packages/utils`, `packages/types`) holding code reused across multiple apps in the same monorepo, consumed via the workspace link rather than duplicated per app or published externally (though they can also be published — see Q9). This gives a single source of truth and lets a change to a shared component and all its consumers land atomically in one commit/PR, avoiding version-mismatch drift between apps.

Structuring guidance: keep each library focused on one responsibility so the dependency graph stays shallow and "affected" detection stays precise (a giant do-everything shared package makes nearly every change "affect" the whole repo); expose a clear public entry point (e.g. `index.ts`) rather than letting consumers deep-import internal files; and use module-boundary lint rules (Nx) or plain code review to prevent circular or architecturally-forbidden imports between library types (e.g. `feature` importing another `feature` directly instead of going through a shared `data-access` layer).

#### Follow-up Questions

- How do you decide when a piece of code deserves its own shared package versus living in a general `utils` folder?
- How do you keep a "shared"/"common" package from becoming a dumping ground that couples unrelated apps together?

### Q9. How do you publish packages from a monorepo to a registry like npm?

#### Answer

Common approaches:

- **Changesets** (or Lerna's versioning): a contributor adds a changeset file describing what changed and the semver bump it warrants; a release job later consumes all pending changesets to bump versions, update changelogs, and `npm publish` each changed package.
- **Nx release** (`nx release`) does the same job graph-aware, publishing only packages that actually changed (and, where needed, their dependents).
- Whichever tool is used, internal `workspace:*` version specifiers must be **rewritten to real semver ranges** at publish time, since a consumer outside the monorepo has no workspace to resolve `workspace:*` against.
- CI typically gates the publish step behind the affected build/tests passing on the merge to the main branch, so nothing broken reaches the registry.

#### Follow-up Questions

- Why can't `workspace:*` be published as-is, and what does the tool substitute it with?
- With independently-versioned packages, how do you make sure package A's published dependency range on package B stays correct across releases?

## Common Pitfalls

- Assuming workspace hoisting behaves like a flat `node_modules` — a package can accidentally rely on a "phantom" dependency it never declared, which then breaks the moment it's extracted from the monorepo or the hoisting layout changes.
- Misconfiguring a task's cache `outputs`/inputs in `turbo.json`/`project.json`, so caching either misses real output files or restores stale results because a relevant input wasn't part of the hash.
- Letting circular dependencies form between shared libraries, which breaks graph-based task ordering and affected detection.
- Publishing a package with an unrewritten `workspace:*` dependency range, which is meaningless outside the monorepo.
- Letting a "shared"/"common" package grow into a dependency bottleneck, so nearly every change is computed as affecting the entire graph.
- Skipping affected-based commands in CI (running full `build`/`test` on every change instead of `nx affected`/`turbo --filter=...[base]`), losing most of the tool's benefit.

## References

-
