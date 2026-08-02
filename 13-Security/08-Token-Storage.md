# Token Storage

> Part of: 13-Security

## Overview

Once a client holds an access token (and possibly a refresh token, from an
[OAuth](06-OAuth.md)/[OIDC](07-OIDC.md) flow or a plain [JWT](05-JWT.md)
session), it has to be kept *somewhere* in the browser between requests.
Every storage option trades off differently between [XSS](02-XSS.md)
exposure (can injected JS read/steal it?) and [CSRF](03-CSRF.md) exposure
(can another site make the browser send it automatically?). There is no
storage location immune to both — the choice is about which risk you can
mitigate more effectively for your app.

## Key Concepts

- **`localStorage`/`sessionStorage`** — plain JS-accessible storage.
  Convenient (no server cookie plumbing, easy to attach to an
  `Authorization` header manually), and not sent automatically so it's not
  CSRF-exposed. But **any XSS bug on the page can read it directly**
  (`localStorage.getItem(...)`), and there's no way to mark it
  script-inaccessible — it's the highest XSS blast radius of the common
  options.
- **Cookies (`HttpOnly`, `Secure`, `SameSite`)** — set by the server via
  `Set-Cookie`, invisible to JS when `HttpOnly` is set (so an XSS bug can't
  read the token directly via `document.cookie`), sent automatically by
  the browser to the matching domain. That automatic attachment is exactly
  what makes cookies CSRF-exposed, mitigated with `SameSite` and/or a CSRF
  token.
  - `HttpOnly` — blocks JS access entirely (`document.cookie` won't show
    it); the main defense against XSS-based token theft.
  - `Secure` — cookie is only ever sent over HTTPS.
  - `SameSite=Strict` — cookie is withheld on all cross-site requests,
    including top-level navigations from an external link (can break flows
    like clicking an email link into a logged-in page on the first
    request).
  - `SameSite=Lax` (default in modern browsers if unset) — cookie is
    withheld on cross-site subresource/POST requests but still sent on
    top-level GET navigations (e.g. following a link), balancing CSRF
    protection with not breaking normal cross-site linking.
  - `SameSite=None` — cookie sent on all cross-site requests; must be
    paired with `Secure`, needed for legitimate cross-site/embedded use
    cases (e.g. a third-party widget), and requires other CSRF defenses.
- **In-memory storage** — keeping the access token only in a JS variable
  (module-level state, not `localStorage`), never persisted. Not readable
  via `document.cookie` or storage APIs by a *different* script context,
  but still readable by any script running in the *same* page context, so
  it doesn't eliminate XSS exposure — it mainly avoids **persistence**
  (survives a page reload) and cross-tab leakage. Commonly paired with a
  `HttpOnly` refresh-token cookie so a page reload can silently mint a new
  access token instead of forcing re-login.
- **Refresh tokens** — long-lived credential used to obtain new
  short-lived access tokens without forcing the user to log in again.
  Because compromising a refresh token has a much larger blast radius than
  compromising an access token (it can mint tokens indefinitely), it's
  typically stored as an `HttpOnly`, `Secure` cookie scoped tightly to the
  token-refresh endpoint's path, and is the primary reason short-lived
  access tokens are practical without hurting UX.
- **Refresh token rotation** — each use of a refresh token issues a *new*
  refresh token and invalidates the old one; if a refresh token is reused
  after having already been rotated (a sign it was stolen and used by
  someone else), the server can detect the reuse and revoke the entire
  token family, limiting how long a stolen refresh token stays useful.
- **The common "best of both" pattern** — short-lived access token kept in
  memory (or `sessionStorage` if it must survive a reload without a
  refresh call), paired with a long-lived refresh token in an `HttpOnly`,
  `Secure`, `SameSite=Strict/Lax` cookie scoped to the refresh endpoint
  only. This limits an XSS bug's reach to the current tab and the access
  token's short lifetime, while the far more sensitive refresh token stays
  out of JS reach entirely.

## Interview Questions & Answers

### Q1. Cookie vs `localStorage` for storing an auth token — which would you pick and why?

#### Answer

Neither is unconditionally safe; the choice is about which attack you can
mitigate better:

- **`localStorage`** is immune to CSRF (nothing is sent automatically) but
  fully exposed to XSS — any injected script can read it and exfiltrate it,
  and there's no flag to make it script-inaccessible.
- An **`HttpOnly` cookie** can't be read by injected JS at all, closing off
  direct token theft via XSS, but it's sent automatically by the browser,
  so it needs `SameSite` and/or CSRF tokens to stop forged cross-site
  requests from riding on it.

In practice, XSS is generally considered the harder-to-fully-eliminate risk
(one missed `dangerouslySetInnerHTML`/third-party script anywhere in the
app is enough), while CSRF is well-mitigated today by `SameSite=Lax/Strict`
being close to a solved problem. That's why `HttpOnly` cookies (or an
in-memory access token backed by an `HttpOnly` refresh cookie) are usually
recommended over `localStorage` for session/refresh tokens.

#### Follow-up Questions

- If your app already has a CSP that blocks all inline scripts, does that
  change this trade-off?
- Why is `sessionStorage` slightly better than `localStorage` here, and
  what does it still not protect against?

### Q2. Why use short-lived access tokens with a refresh token instead of one long-lived token?

#### Answer

A single long-lived token means a leak (via XSS, a logged request, a
compromised device) stays valid and unrevoked for its entire lifetime,
since stateless tokens can't be individually invalidated without extra
infrastructure. Splitting into a short-lived access token (minutes) plus a
long-lived refresh token limits exposure: a stolen access token expires
quickly on its own, while the more sensitive refresh token is kept in a
much more restricted place (`HttpOnly` cookie, restricted path) and is
used rarely enough that rotation/reuse-detection can catch theft — if a
refresh token is used twice (once by the attacker, once by the legitimate
client), the server can detect that mismatch and revoke the whole session
family immediately.

#### Follow-up Questions

- What UX problem does silent-refresh-on-reload solve, and what's the
  trade-off of doing that refresh call on every page load?
- How would refresh token rotation detect that a token was stolen, and
  what should the server do when it detects reuse?

## Common Pitfalls

- Storing a long-lived JWT in `localStorage` "for simplicity," turning any
  future XSS bug into full, silent, long-duration session takeover.
- Setting `SameSite=None` on a session cookie without a compensating CSRF
  token, "because some third-party redirect broke otherwise."
- Storing the refresh token in the same place/scope as the access token —
  it should be more tightly scoped (`HttpOnly`, restricted `Path`, ideally
  its own subdomain/endpoint) since it's the more valuable credential.
- Not implementing refresh token rotation/reuse detection, so a
  once-stolen refresh token remains silently usable for its entire
  lifetime.
- Assuming an in-memory token is "safe from XSS" — it still isn't;
  it's just not persisted or automatically shared across tabs, which
  reduces exposure but does not eliminate it.

## References

- [OWASP: HTML5 Security Cheat Sheet — Local Storage](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#local-storage)
- [MDN: Set-Cookie — SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [Auth0: Refresh Token Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [RFC 6265: HTTP State Management Mechanism (Cookies)](https://datatracker.ietf.org/doc/html/rfc6265)
