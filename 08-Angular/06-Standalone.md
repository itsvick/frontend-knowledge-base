# Standalone

> Part of: 08-Angular

## Overview

Standalone components (introduced in Angular 14, the default since Angular 19) let a component, directive, or pipe declare its own dependencies directly, without being registered in an `NgModule`.

## Key Concepts

- A component opts in with `standalone: true` (implicit/default from Angular 19 onward).
- Dependencies (other components, directives, pipes, `CommonModule`, etc.) are listed in the component's own `imports` array instead of an `NgModule`'s.
- Apps can bootstrap without a root module via `bootstrapApplication()` instead of `platformBrowserDynamic().bootstrapModule()`.
- Routes can lazy-load a standalone component directly with `loadComponent()`, without a lazy-loaded feature module.
- Standalone and `NgModule`-based components can coexist, which is what makes incremental migration possible.

## Interview Questions & Answers

### Q1. What are Standalone Components?

#### Answer

Standalone components are Angular components (also applies to directives and pipes) that manage their own dependencies via an `imports` array, removing the need to declare them inside an `NgModule`. This simplifies the component model, reduces boilerplate (no more `SharedModule`/`FeatureModule` wiring just to export a component), and makes lazy loading more granular since individual components — not just modules — can be lazy-loaded.

#### Code Example

```ts
@Component({
  selector: "app-user-card",
  standalone: true,
  imports: [CommonModule, RouterLink],
  template: `<div *ngIf="user">{{ user.name }}</div>`,
})
class UserCardComponent {
  @Input() user?: User;
}

// bootstrapping without a root NgModule
bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)],
});

// lazy-loading a standalone component directly in a route
const routes: Routes = [
  { path: "profile", loadComponent: () => import("./profile.component").then((m) => m.ProfileComponent) },
];
```

#### Follow-up Questions

- How do standalone components change how you structure lazy-loaded feature areas?
- Can a standalone component still be declared inside an `NgModule`?

## Common Pitfalls

- Forgetting to import a dependency (e.g. `CommonModule` for `*ngIf`) directly into the standalone component, since it no longer inherits it from a shared module.
- Assuming standalone components can't interoperate with existing `NgModule`-based code — they can, in both directions.
