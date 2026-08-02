# Browser

> Part of: 03-HTML

## Overview

The DOM is how the browser represents an HTML document to JavaScript, events (with bubbling/delegation) are how the browser reports user interaction with that document, and `localStorage`/`sessionStorage`/cookies are the browser's built-in mechanisms for persisting data client-side between requests.

## Key Concepts

- DOM: tree-of-objects representation of an HTML document that JS can read/manipulate
- Event bubbling: an event propagates from the target element up through its ancestors
- Event delegation: attach one listener on a parent to handle events for its (current or future) children, relying on bubbling
- Client-side storage: `localStorage`, `sessionStorage`, and cookies all persist data in the browser, but differ in lifetime, scope, capacity, and whether the server can see them

## Interview Questions & Answers

### Q1. What is the DOM?

#### Answer

The Document Object Model (DOM) represents an HTML document as a tree of objects that JavaScript can manipulate.

### Q2. What is event bubbling?

#### Answer

An event starts from the target element and propagates upward through its parent elements.

### Q3. What is event delegation and why does it matter?

#### Answer

Event delegation means attaching a single event listener to a parent element instead of one listener per child, relying on event bubbling to catch events as they propagate up from whichever child was actually interacted with.

This matters because it:
- **Reduces memory usage** — one listener instead of potentially thousands (e.g. one per `<li>` in a large list).
- **Improves performance** — fewer listener registrations, especially on pages that render many items.
- **Works for dynamic elements** — children added to the parent *after* the listener was attached are still handled, since the event still bubbles up to the parent; per-child listeners would miss them entirely unless re-attached on every update.

This makes it especially common for large or dynamically-updated lists.

#### Code Example

```js
// Event delegation: one listener on the parent handles clicks on any current or future <li>
document.querySelector("ul").addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log("Clicked:", e.target.textContent);
  }
});
```

### Q4. What is the difference between `localStorage`, `sessionStorage`, and cookies?

#### Answer

All three store data in the browser, but they differ in how long the data lives, what can access it, and how much you can store:

| | `localStorage` | `sessionStorage` | Cookies |
|---|---|---|---|
| Lifetime | Until manually cleared | Until the tab is closed | Until expiry (or session, if none set) |
| Scope | Per origin, shared across tabs | Per origin **and** per tab (not shared between tabs, even for the same site) | Per domain (+ path), sent with matching requests |
| Capacity | ~5–10MB | ~5–10MB | ~4KB |
| Sent to server? | No | No | Yes — automatically, with every matching HTTP request |
| JS-accessible? | Yes | Yes | Yes, unless `HttpOnly` is set |

- **`localStorage`** — persists indefinitely (survives closing the browser), scoped to the origin, good for long-lived data like theme or language preference. Values are always stored as strings.
  ```js
  localStorage.setItem("theme", "dark");
  ```
- **`sessionStorage`** — same API and origin scoping as `localStorage`, but cleared when the tab closes, and *not* shared across tabs even for the same site/origin — each tab gets its own. Good for short-lived, per-tab state like a multi-step form.
  ```js
  sessionStorage.setItem("formStep", "2");
  ```
- **Cookies** — the only one of the three the server can see, since the browser automatically attaches matching cookies to every HTTP request to that domain. Much smaller (~4KB), can have an expiry, and support flags like `HttpOnly` (inaccessible to JS) and `Secure` (HTTPS-only).
  ```js
  document.cookie = "user=Pranav";
  ```

**When to use which:**
- `localStorage` — long-term client-only data (preferences, non-sensitive cached state).
- `sessionStorage` — temporary, per-tab state that shouldn't outlive the tab.
- Cookies — whenever the *server* needs to see the value on each request (e.g. session/auth state).

Because any script running on the page can read `localStorage`/`sessionStorage`, storing sensitive values like auth tokens there is vulnerable to XSS-based theft; an `HttpOnly` cookie is generally the safer choice for authentication since it's invisible to JS. See [Token Storage](../13-Security/08-Token-Storage.md) for the fuller auth-token trade-off (XSS vs CSRF).

#### Follow-up Questions

- Why doesn't `sessionStorage` get shared between two tabs open to the same site, even though `localStorage` is shared?
- What happens to a cookie's `Secure`/`HttpOnly`/`SameSite` flags if you set it via `document.cookie` instead of a `Set-Cookie` response header?
- Given cookies are sent with every request, what's the performance implication of storing a lot of data in one?

## Common Pitfalls

- Attaching a listener to every child element instead of delegating to a parent, which misses dynamically added children and hurts performance.
- Forgetting `e.stopPropagation()` when a bubbling event should not reach an ancestor's handler.
- Storing sensitive data (auth tokens) in `localStorage`/`sessionStorage`, where any injected script (XSS) can read it directly — see [Token Storage](../13-Security/08-Token-Storage.md).
- Assuming `sessionStorage` is shared across tabs like `localStorage` is — it isn't, which breaks state expected to persist when a user opens a link in a new tab.
