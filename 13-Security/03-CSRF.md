# CSRF

> Part of: 13-Security

## Overview

Cross-Site Request Forgery (CSRF) tricks a victim's browser into submitting
a state-changing request (e.g. transfer funds, change email) to a site
where the victim is already authenticated, without the victim's knowledge.
It exploits the fact that browsers automatically attach cookies to any
request to that cookie's domain — including requests initiated by a
completely different, malicious site.

## Key Concepts

- **Ambient authority** — the browser sends cookies automatically with
  every request to the matching domain, regardless of which page/origin
  triggered the request. CSRF abuses this: the attacker's page doesn't need
  to read the victim's cookie, it just needs the browser to attach it.
- **Requires a session riding on cookies** — CSRF specifically targets
  cookie-based auth. Requests authenticated via a bearer token that the
  attacker's page can't read (e.g. an `Authorization` header set from
  memory/JS) aren't vulnerable in the same way, because a cross-origin page
  can't force your browser to attach a custom header it doesn't know.
- **Same-origin policy vs CSRF** — SOP stops the attacker's page from
  *reading* the response of a cross-origin request, but it does **not**
  stop the browser from *sending* the request in the first place (for
  simple GET/POST forms). CSRF exploits that gap.
- **Common defenses**:
  - **CSRF (synchronizer) tokens** — server embeds a random, unpredictable
    token in the legitimate form/page; the request is only accepted if that
    token is present and matches, which an attacker's cross-origin page
    can't obtain.
  - **`SameSite` cookies** — `SameSite=Lax` or `Strict` tell the browser not
    to send the cookie on cross-site requests, closing most CSRF vectors at
    the browser level (see [SameSite cookies](08-Token-Storage.md)).
  - **Custom headers** — requiring a custom header (e.g.
    `X-Requested-With`) on state-changing requests works as a CSRF defense
    because cross-origin `<form>` submissions can't set custom headers, and
    `fetch`/XHR from another origin would be blocked by CORS before the
    header is even accepted (assuming the server doesn't allow the
    attacker's origin).
  - **Double-submit cookie pattern** — server sets a random token both as a
    cookie and expects it echoed back in a request header/body; since a
    cross-origin attacker can't read the cookie's value (no access to
    `document.cookie` for another domain), they can't produce a matching
    value.
- **CSRF vs XSS** — CSRF forges a request as the victim without needing to
  execute any code in the victim's origin; XSS executes attacker code
  *inside* the victim's origin (and can be used to bypass CSRF tokens by
  reading them directly, since same-origin script has full access). They're
  complementary, not the same bug.

## Interview Questions & Answers

### Q1. What is CSRF and why do browsers allow it to happen?

#### Answer

CSRF forges a state-changing request from an authenticated victim's
browser by having them (unknowingly) submit a request to another site —
usually via an auto-submitting form or an `<img>`/`fetch` on a malicious
page. It works because browsers attach cookies to any request to a domain
automatically, regardless of which page initiated the request; the server
sees a normal, cookie-authenticated request and can't tell it wasn't
intentionally triggered by the user.

#### Follow-up Questions

- Why doesn't CORS prevent CSRF?
- Would a login page itself typically need CSRF protection? Why or why not?

### Q2. How do you protect an app against CSRF?

#### Answer

- Prefer `SameSite=Lax` (or `Strict` for highly sensitive actions) on
  session cookies — this alone blocks most real-world CSRF since modern
  browsers default new cookies to `Lax` if unset.
- Use anti-CSRF tokens for state-changing requests (POST/PUT/DELETE):
  generate a per-session (or per-request) token, embed it in forms/headers,
  and validate it server-side before applying the mutation.
- For SPA/API architectures, avoid relying on cookies for auth where
  possible — authenticate via a bearer token attached manually in an
  `Authorization` header (not auto-attached by the browser), which sidesteps
  classic CSRF entirely, at the cost of needing your own XSS-safe storage
  strategy for that token.
- Require re-authentication (password/OTP) for highly sensitive actions
  (changing email/password, payments) as a last line of defense.
- Set CORS correctly on the API — it won't stop simple form-based CSRF, but
  it prevents malicious pages from reading responses to forged `fetch`
  requests, and blocks credentialed cross-origin `fetch` unless the origin
  is explicitly allow-listed with `Access-Control-Allow-Credentials: true`.

#### Code Example

```html
<!-- Attacker's page: auto-submits a transfer request using the victim's
     existing session cookie for bank.com -->
<form action="https://bank.com/transfer" method="POST" id="csrf">
  <input type="hidden" name="to" value="attacker-account" />
  <input type="hidden" name="amount" value="10000" />
</form>
<script>document.getElementById('csrf').submit();</script>
```

```js
// Server-side mitigation: reject unless a matching CSRF token is present
app.post('/transfer', (req, res) => {
  if (req.body.csrfToken !== req.session.csrfToken) {
    return res.status(403).send('Invalid CSRF token');
  }
  // ...proceed with transfer
});
```

#### Follow-up Questions

- If a session cookie is `SameSite=Strict`, are you fully protected from
  CSRF? What edge cases remain (e.g. top-level navigations, subdomains)?
- How would double-submit cookies fail if the app also has an XSS bug?

## Common Pitfalls

- Believing CORS configuration alone prevents CSRF — CORS governs whether
  a script can *read* a cross-origin response, not whether the browser
  *sends* the request.
- Using `SameSite=None` without also using `Secure` and a CSRF token,
  because some third-party embed/redirect flow "needs" cross-site cookies.
- Putting the CSRF token in a cookie only (no server-side validation
  against a session-bound value) — that's just a cookie, not a token pair.
- Forgetting that GET requests should never cause state changes; if they
  do, they're trivially CSRF-able via a plain `<img src="...">`.

## References

- [OWASP: Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [MDN: SameSite cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
