# HTTP Interceptors

> Part of: 08-Angular

## Overview

HTTP interceptors let you intercept and transform outgoing `HttpClient` requests and incoming responses in one central place, instead of repeating the same logic at every call site.

## Key Concepts

- An interceptor sits between `HttpClient` and the backend, seeing every request/response that passes through.
- Class-based interceptors implement `HttpInterceptor` (`intercept(req, next)`); functional interceptors (Angular 15+) are plain functions (`HttpInterceptorFn`) registered via `withInterceptors()`.
- Interceptors are chained — each one calls `next.handle(req)` (or `next(req)` for functional) to pass control to the next interceptor/backend, and can transform the request before calling it or the response after.
- Common use cases: attaching auth tokens, global error handling, loading indicators, logging, retry logic, response caching.
- Requests are immutable — an interceptor clones the request (`req.clone({...})`) to modify headers/params rather than mutating it directly.

## Interview Questions & Answers

### Q1. What are Angular HTTP Interceptors?

#### Answer

An HTTP interceptor is a piece of code that sits in the pipeline between `HttpClient` and the actual network call, able to inspect and transform every outgoing request and every incoming response (or error). Instead of adding the same logic (like an auth header, or error handling) into every service that makes an HTTP call, you register an interceptor once and it applies globally.

Angular supports two styles: class-based interceptors that implement `HttpInterceptor` and are registered via `HTTP_INTERCEPTORS` in providers, and functional interceptors (a plain function matching `HttpInterceptorFn`) registered via `provideHttpClient(withInterceptors([...]))`, which is the current recommended approach for standalone apps.

#### Code Example

```ts
// Functional interceptor — attaches an auth token to every outgoing request
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  const token = auth.getToken();
  const authReq = token
    ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
    : req;
  return next(authReq);
};

// Registered in app config
bootstrapApplication(AppComponent, {
  providers: [provideHttpClient(withInterceptors([authInterceptor]))],
});
```

#### Follow-up Questions

- How do multiple interceptors compose, and does registration order matter?
- How would you use an interceptor to implement a global loading spinner or centralized error handling?

### Q2. Have you implemented an Interceptor? What was your real-time use case?

#### Answer

A typical real-world example: an **auth interceptor** that attaches the current access token as an `Authorization` header to every outgoing API request, so individual services never need to know about the token. Paired with that, an **error-handling interceptor** catches `401 Unauthorized` responses globally — on a 401 it triggers a token refresh (or logs the user out and redirects to `/login`) instead of every single API call needing its own error handling for expired sessions. Other common production use cases include a **loading interceptor** that increments/decrements a shared "in-flight requests" counter to drive a global spinner, and a **logging interceptor** that records request/response timing for diagnostics.

#### Follow-up Questions

- How do you avoid an infinite loop when a token-refresh call itself goes through the same interceptor?

## Common Pitfalls

- Mutating the request object directly instead of using `req.clone()` — `HttpRequest` is immutable and mutation silently has no effect.
- Forgetting that interceptor order matters — e.g. a logging interceptor registered before an auth interceptor won't see the auth header that gets added afterward.
- Not excluding the interceptor's own dependent calls (like a token-refresh request) from itself, causing infinite recursion.
