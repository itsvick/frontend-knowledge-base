# 08-Angular — FAQs

> Quick-fire Q&A for 08-Angular. Keep answers short (2-5 lines); link to the relevant topic file for depth.

## Q1. What is the difference between `dependencies` and `devDependencies` in `package.json`?

**A:** `dependencies` lists packages required at runtime (e.g. `@angular/core`, `rxjs`) — they ship as part of the built application. `devDependencies` lists packages only needed during development/build (e.g. `@angular/cli`, TypeScript, testing/linting tools) — they aren't needed by the running app and are typically skipped with `npm install --production`.

## Q2. What is the best place/pattern to make API calls in Angular?

**A:** API calls belong in injectable services (not components) using `HttpClient`, keeping components free of HTTP/data-fetching concerns. Combine with HTTP interceptors (see [09-HTTP-Interceptors.md](./09-HTTP-Interceptors.md)) for cross-cutting concerns like auth headers or error handling instead of repeating them per call. Use RxJS operators like `switchMap` inside a service method to auto-cancel a stale in-flight request when a new one starts (e.g. typeahead search). Handle errors with `catchError` inside the service or a shared interceptor rather than scattering try/catch-equivalents across components.

## Q3. How would you improve the performance of an Angular application?

**A:** Use `OnPush` change detection where possible (see [03-Change-Detection.md](./03-Change-Detection.md)) so components only re-render on input reference changes, and add a `trackBy` function to `*ngFor` to avoid full DOM re-creation on list updates. Lazy-load feature routes (see [05-Routing.md](./05-Routing.md)) so the initial bundle only ships what's needed, and prefer pure pipes or precomputed values over calling methods directly in templates — a template method call re-executes on every change detection cycle. Use the `async` pipe with observables so subscriptions are managed and cleaned up automatically, use CDK virtual scrolling for long lists, and always unsubscribe manual `.subscribe()` calls (via `takeUntil`, `async` pipe, or Angular's `DestroyRef`) to avoid memory leaks that also indirectly hurt performance over time.

## Q4. What are the key Angular-specific security practices (XSS, DomSanitizer, CSP, Route Guards)?

**A:** Angular auto-escapes interpolated/bound values in templates by default, which prevents most XSS without extra work. `DomSanitizer`'s `bypassSecurityTrustHtml`/`bypassSecurityTrustUrl`/etc. exist for the rare case you deliberately need to render trusted-but-otherwise-blocked content (e.g. a CMS-provided HTML snippet) — using it carelessly on user-controlled input reopens the XSS hole Angular closes by default, so it should only wrap content you trust. A Content-Security-Policy header restricts what scripts/styles/resources the page may load, providing defense-in-depth even if an XSS bug slips through. Route Guards (`CanActivate`/`CanMatch`, see [05-Routing.md](./05-Routing.md)) control page-level authorization — they stop unauthorized navigation but are not a substitute for server-side authorization, since client-side guards can be bypassed by a determined user hitting the API directly.
