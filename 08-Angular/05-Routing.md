# Routing

> Part of: 08-Angular

## Overview

Angular's Router handles navigation between views. Lazy loading and route guards are two of its most interview-relevant features: one controls *when* code is loaded, the other controls *whether* navigation is allowed to proceed.

## Key Concepts

- Lazy loading defers loading a feature's code until its route is actually navigated to, via `loadChildren` (module) or `loadComponent` (standalone component).
- Lazy loading reduces the initial bundle size and speeds up first load, at the cost of a small delay the first time a lazy route is visited.
- Route guards are functions/classes that run at specific points in the navigation lifecycle and can allow, block, or redirect navigation.
- Modern Angular favors functional guards (plain functions using `inject()`) over the older class-based `CanActivate` interface, though both work.

## Interview Questions & Answers

### Q1. What is Lazy Loading? Why do we use it?

#### Answer

Lazy loading is the practice of loading a feature's JavaScript only when the user actually navigates to a route that needs it, instead of bundling everything into the initial app load. In Angular this is configured on a route with `loadChildren` (for an `NgModule`) or `loadComponent` (for a standalone component), both of which take a dynamic `import()`. The router splits that code into a separate chunk that's fetched on demand.

We use it to keep the initial bundle small, which speeds up first paint and time-to-interactive — especially important for large applications where most users only ever touch a subset of the features in a single session.

#### Code Example

```ts
const routes: Routes = [
  {
    path: "admin",
    loadChildren: () => import("./admin/admin.module").then((m) => m.AdminModule),
  },
  {
    path: "profile",
    loadComponent: () => import("./profile/profile.component").then((m) => m.ProfileComponent),
  },
];
```

#### Follow-up Questions

- How does lazy loading interact with route guards on the same route?
- What's the tradeoff of lazy loading too aggressively (e.g. splitting every single route)?

### Q2. What are Route Guards? Explain each type (`CanActivate`, `CanDeactivate`, `CanLoad`, `CanMatch`, `Resolve`, etc.).

#### Answer

Route guards are functions that the router runs at specific points during navigation to decide whether to allow it, redirect it, or block it. Each guard type hooks into a different point in the navigation lifecycle:

- **`CanActivate`** — runs before activating a route; return `false`/a `UrlTree` to block/redirect (e.g. checking if a user is authenticated).
- **`CanActivateChild`** — same as `CanActivate`, but applied to a route's child routes.
- **`CanDeactivate`** — runs before leaving a route; commonly used to warn about unsaved changes before navigating away.
- **`CanLoad`** (deprecated in favor of `CanMatch`) — used to prevent lazy-loaded module code from being fetched at all if the guard fails.
- **`CanMatch`** — decides whether a route config is even considered a match for a URL, allowing multiple routes to share the same path and letting the router fall through to the next matching route if the guard fails (this replaces `CanLoad` and also gates lazy-loaded code from being fetched).
- **`Resolve`** — pre-fetches data needed by a route before activating it, so the component doesn't render in a loading state; the resolved data is available via `ActivatedRoute.data`.

#### Code Example

```ts
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() ? true : router.createUrlTree(["/login"]);
};

const routes: Routes = [
  {
    path: "dashboard",
    canActivate: [authGuard],
    canMatch: [featureFlagGuard],
    resolve: { user: userResolver },
    loadComponent: () => import("./dashboard.component").then((m) => m.DashboardComponent),
  },
];
```

#### Follow-up Questions

- Why was `CanLoad` deprecated in favor of `CanMatch`?
- How would you implement an "unsaved changes" confirmation using `CanDeactivate`?

## Common Pitfalls

- Using `CanActivate` when the intent is really to prevent code from being fetched at all — that's what `CanMatch` (or the deprecated `CanLoad`) is for.
- Forgetting that a `Resolve` guard that never completes (e.g. an observable that doesn't emit) blocks navigation indefinitely.
- Lazy-loading every route individually, creating excessive small chunks and more round trips than necessary.
