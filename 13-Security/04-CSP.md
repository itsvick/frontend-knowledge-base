# CSP

> Part of: 13-Security

## Overview

Content Security Policy (CSP) is an HTTP response header (or `<meta>` tag)
that tells the browser which sources of scripts, styles, images, fonts,
frames, etc. are allowed to load/execute on a page. It's a defense-in-depth
layer against [XSS](02-XSS.md) and data-injection attacks: even if an
attacker manages to inject a `<script>` tag, a strict CSP can stop it from
executing because its origin isn't on the allow-list.

## Key Concepts

- **Directive-based allow-listing** — CSP is a set of directives, each
  scoping what's allowed for one resource type: `script-src`, `style-src`,
  `img-src`, `connect-src` (fetch/XHR/WebSocket targets), `frame-ancestors`
  (who can iframe this page, replacing the older `X-Frame-Options`),
  `default-src` (fallback for unlisted directives), etc.
- **`'unsafe-inline'` and `'unsafe-eval'`** — opt back into inline
  `<script>`/`onclick=` handlers or `eval()`/`new Function()`, which
  defeats most of CSP's XSS protection. A strict policy omits both, which
  forces inline scripts/styles to move to external files or use a
  nonce/hash.
- **Nonces and hashes** — instead of `'unsafe-inline'`, generate a random
  per-response nonce (`script-src 'nonce-<random>'`) and add it as an
  attribute on each legitimate `<script>` tag, or allow-list the exact hash
  of a known inline script's content. Only scripts with the matching
  nonce/hash execute; attacker-injected `<script>` tags won't have it.
- **Report-only mode** — `Content-Security-Policy-Report-Only` lets you
  roll out a policy and collect violation reports (via `report-uri`/
  `report-to`) without actually blocking anything yet, useful for safely
  tightening a policy in production before enforcing it.
- **Enforced via the `Content-Security-Policy` header** (or `<meta
  http-equiv="Content-Security-Policy">`, with some directives like
  `frame-ancestors` and `report-uri` not supported in the meta form) — the
  browser parses it once per response and enforces it for that document.
- **Not a substitute for output encoding** — CSP reduces the *impact* of an
  XSS bug (blocks the injected script from running/exfiltrating data via
  `connect-src`), but proper escaping/sanitization is still the primary
  defense; CSP is the safety net, not the first line.

## Interview Questions & Answers

### Q1. What is CSP and how does it help mitigate XSS?

#### Answer

CSP is a header-driven allow-list that restricts which origins a page may
load scripts, styles, images, frames, and other resources from. If an
attacker manages to inject a `<script src="https://evil.com/x.js">` or an
inline `<script>` via an XSS bug, a strict policy (e.g. `script-src 'self'`
with no `'unsafe-inline'`) makes the browser refuse to execute it, because
`evil.com` isn't allow-listed and inline scripts aren't permitted at all.
It doesn't prevent the injection itself, but it stops the injected code
from doing anything.

#### Follow-up Questions

- Why is `script-src 'self' 'unsafe-inline'` considered a weak policy
  despite restricting external domains?
- How would you introduce CSP into a legacy app that currently relies
  heavily on inline `<script>` tags without breaking it?

### Q2. How would you write and roll out a strict CSP for a React SPA?

#### Answer

- Start from `default-src 'self'` and add only the specific directives you
  need (e.g. `connect-src` for your API origin, `img-src` for a CDN).
- Avoid `'unsafe-inline'`/`'unsafe-eval'`; use a per-request nonce injected
  into your build's script tags instead of inline scripts.
- Roll out with `Content-Security-Policy-Report-Only` first, wire up a
  `report-uri`/`report-to` endpoint, monitor violations for a rollout
  period, fix legitimate breakage, then switch to the enforcing header.
- Set `frame-ancestors 'none'` (or a specific allow-list) to prevent
  clickjacking via iframe embedding.
- Keep the policy as an explicit allow-list rather than trying to
  block-list known-bad domains — allow-lists degrade much more gracefully.

#### Code Example

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-r4nd0m123';
  style-src 'self' 'nonce-r4nd0m123';
  img-src 'self' https://cdn.example.com;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
  report-to csp-endpoint;
```

```html
<!-- Server generates a fresh nonce per response and injects it here -->
<script nonce="r4nd0m123" src="/bundle.js"></script>
```

#### Follow-up Questions

- Why must the nonce be regenerated on every response rather than reused?
- What's the trade-off between hash-based and nonce-based CSP for scripts
  bundled by a build tool?

## Common Pitfalls

- Shipping `script-src 'unsafe-inline'` "temporarily" to unblock a rollout
  and never removing it — it neutralizes CSP's main XSS protection.
- Using CSP as the *only* XSS defense and skipping output
  encoding/sanitization, on the assumption "the browser will block it
  anyway."
- Forgetting `frame-ancestors` and remaining vulnerable to clickjacking
  even with a strict `script-src`.
- Setting the policy via `<meta>` tag and expecting `report-uri` or
  `frame-ancestors` to work — both require the real HTTP header.

## References

- [MDN: Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
- [OWASP: Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [web.dev: Mitigate XSS with CSP](https://web.dev/articles/csp)
