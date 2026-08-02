# XSS

> Part of: 13-Security

## Overview

Cross-Site Scripting (XSS) is a class of injection vulnerability where an
attacker gets untrusted, attacker-controlled data executed as script in a
victim's browser, in the origin of a trusted site. Because the script runs
under the victim's own session/origin, it can read cookies (if not
`HttpOnly`), make authenticated requests, steal tokens, deface the page, or
pivot into further attacks (e.g. CSRF, session hijacking).

## Key Concepts

- **Stored XSS** — malicious script is persisted server-side (DB, comment
  field, profile bio) and served to every visitor who views that content.
  Highest impact since it's automatic and repeatable.
- **Reflected XSS** — payload is part of the request (query param, form
  field) and is echoed back unescaped in the response. Requires tricking the
  victim into clicking a crafted link.
- **DOM-based XSS** — the vulnerability lives entirely client-side: JS reads
  attacker-controlled data (URL, `location.hash`, `postMessage`) and writes
  it into the DOM via a dangerous sink (`innerHTML`, `document.write`,
  `eval`) without the payload ever touching the server.
- **Sources vs sinks** — a *source* is anywhere attacker-controlled data
  enters (URL, form input, API response); a *sink* is anywhere it's
  interpreted as code/markup (`innerHTML`, `eval`, `href`/`src` attributes,
  `dangerouslySetInnerHTML`). XSS = untrusted data flowing from a source to
  a sink without proper encoding/sanitization.
- **Escaping/encoding vs sanitization** — escaping (e.g. HTML-entity
  encoding `<` → `&lt;`) prevents data from being *parsed* as markup at all;
  sanitization (e.g. via DOMPurify) allows a restricted, safe subset of
  HTML/markup through after stripping dangerous tags/attributes. Use
  escaping by default for plain text; use a sanitizer only when you
  genuinely need to render user-supplied HTML.
- **Framework auto-escaping** — React (`{value}` in JSX), Angular, and Vue
  all escape interpolated values by default. XSS bugs in these frameworks
  almost always come from explicitly opting out of that protection
  (`dangerouslySetInnerHTML`, Angular's `bypassSecurityTrust*`, Vue's
  `v-html`) with unsanitized input.
- **Defense in depth** — output encoding is the primary defense, but a
  strict [Content Security Policy](04-CSP.md), `HttpOnly`/`Secure`/
  `SameSite` cookies, and input validation all reduce the blast radius even
  if one layer fails.

## Interview Questions & Answers

### Q1. What is XSS and what are the three main types?

#### Answer

XSS lets an attacker inject and execute arbitrary JavaScript in the context
of a trusted origin, because the victim's browser can't distinguish the
attacker's script from the site's own code. The three types differ in
*where* the untrusted data is stored/reflected before it reaches a sink:

- **Stored** — persisted on the server and served to other users later
  (e.g. an unsanitized comment field).
- **Reflected** — bounced straight back in the response from request data,
  requires a crafted link/form to trigger.
- **DOM-based** — never touches the server; client-side JS pipes
  attacker-controlled data (URL, hash) into a dangerous DOM sink.

#### Follow-up Questions

- Why is stored XSS generally considered more dangerous than reflected XSS?
- How would you find a DOM-based XSS bug that a server-side scanner would
  miss?

### Q2. How do you prevent XSS in a React/frontend application?

#### Answer

- Rely on JSX's default escaping — never construct HTML strings by
  concatenating user input.
- Avoid `dangerouslySetInnerHTML`; if you must render user-supplied HTML,
  run it through a sanitizer (e.g. DOMPurify) first and keep the allow-list
  of tags/attributes minimal.
- Never put unvalidated user input into `href`, `src`, or `style` attributes
  directly — a `javascript:` URL in `href` is a classic sink.
- Set a strict [CSP](04-CSP.md) (`script-src 'self'`, no `'unsafe-inline'`)
  as a second layer so injected `<script>` tags fail to execute even if
  encoding is missed somewhere.
- Mark auth cookies `HttpOnly` so injected scripts can't read them via
  `document.cookie` even if an XSS bug exists (see
  [Cookie vs LocalStorage](08-Token-Storage.md)).
- Validate/encode on the server too — client-side escaping alone doesn't
  protect API consumers that aren't your React app.

#### Code Example

```jsx
// Vulnerable: renders raw HTML from an untrusted source
function Comment({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// Safer: sanitize before allowing any HTML through
import DOMPurify from 'dompurify';

function Comment({ html }) {
  const clean = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}

// Safest for plain text: let JSX escape it, no dangerouslySetInnerHTML at all
function Comment({ text }) {
  return <div>{text}</div>;
}
```

#### Follow-up Questions

- Why doesn't React's default escaping protect you if you use
  `dangerouslySetInnerHTML`?
- What's the difference between sanitizing on the client vs the server, and
  why do you need both?

## Common Pitfalls

- Assuming a frontend framework's auto-escaping means XSS is "solved" —
  any `dangerouslySetInnerHTML`/`v-html`/raw DOM API call reopens the hole.
- Sanitizing on the client only, then trusting the same data when it's
  rendered elsewhere (native app, email template, admin panel) that reads
  from the same API.
- Rolling a custom regex-based sanitizer instead of a maintained library —
  HTML parsing edge cases (mutation XSS, malformed tags) are notoriously
  easy to get wrong.
- Storing JWTs/session tokens in `localStorage` "because it's simpler" —
  makes any XSS bug into full session theft since JS can read it.

## References

- [OWASP: Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN: Types of attacks - XSS](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks#cross-site_scripting_xss)
- [DOMPurify](https://github.com/cure53/DOMPurify)
