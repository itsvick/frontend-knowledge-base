# OAuth

> Part of: 13-Security

## Overview

OAuth 2.0 is an **authorization** framework: it lets a user grant a
third-party app limited access to their resources on another service
(e.g. "let this app read my Google Calendar") without ever sharing their
password with that app. It defines how an app obtains an access token
scoped to specific permissions, not who the user is — that's what
[OIDC](07-OIDC.md) adds on top.

## Key Concepts

- **Roles** — *Resource Owner* (the user), *Client* (the app requesting
  access), *Authorization Server* (issues tokens, e.g. Google/Auth0),
  *Resource Server* (the API holding the protected data, which validates
  the access token).
- **Access token** — a credential (often a [JWT](05-JWT.md), but can be
  opaque) the client presents to the resource server to access protected
  resources, scoped to specific permissions (`scope`) and normally
  short-lived.
- **Grant types (flows)**:
  - **Authorization Code** — the client redirects the user to the
    authorization server, the user logs in and consents, the server
    redirects back with a one-time `code`, and the client exchanges that
    code (server-side, with a client secret) for an access token. The
    standard flow for server-rendered/confidential clients.
  - **Authorization Code + PKCE** — the same flow, but the client also
    generates a random `code_verifier`, sends its hash (`code_challenge`)
    up front, and must present the original verifier when exchanging the
    code. This proves the app exchanging the code is the same one that
    started the flow, closing the code-interception gap that public
    clients (SPAs, mobile apps — can't safely hold a client secret) would
    otherwise have. **This is now the recommended flow for SPAs and mobile
    apps**, replacing the old Implicit flow.
  - **Implicit** (deprecated) — returned the access token directly in the
    redirect URL fragment, with no code-exchange step, designed for
    JS-only apps that couldn't hold a secret. Deprecated because the token
    ends up in the browser history/referrer and there's no way to bind the
    token to the client, making it more exposed than Auth Code + PKCE.
  - **Client Credentials** — machine-to-machine: the client authenticates
    with its own client ID/secret directly (no user/resource-owner
    involved), used for service-to-service API access.
  - **Resource Owner Password Credentials** (deprecated) — the client
    collects the user's username/password directly and exchanges them for
    a token. Deprecated because it defeats OAuth's core purpose (never
    sharing credentials with the client) and only made sense for
    highly-trusted first-party clients.
- **`state` parameter** — an opaque, unpredictable value the client sends
  in the initial authorization request and verifies on the redirect back,
  preventing CSRF against the OAuth flow itself (an attacker tricking a
  victim into completing *the attacker's* auth flow under the victim's
  session).
- **Scopes** — a space-separated list of permissions the client is
  requesting (`read:calendar`, `write:contacts`); the authorization server
  shows these to the user for consent and encodes them into the issued
  token so the resource server can enforce least privilege.

## Interview Questions & Answers

### Q1. Why was the Implicit flow deprecated in favor of Authorization Code + PKCE for SPAs?

#### Answer

The Implicit flow returns the access token directly in the redirect URL's
fragment after login, with no server-side exchange step — convenient for a
JS-only app since there's no client secret to protect, but it means the
token passes through the browser history, can leak via referrer headers or
browser extensions, and can't be bound to the specific client instance that
requested it (no proof of possession). Authorization Code + PKCE instead
returns a short-lived, single-use `code`; exchanging it for a token
requires the original `code_verifier` that only the legitimate client
generated, so even if the code is intercepted in transit, an attacker can't
redeem it without that verifier. It also means the actual access token
never appears in a browser-visible URL. As a result, current OAuth
guidance (and OAuth 2.1) drops the Implicit flow entirely.

#### Follow-up Questions

- How does PKCE protect against a malicious app on the same device
  intercepting the authorization code via a custom URL scheme?
- Where would you store the access token once the SPA receives it from the
  code exchange? (see [Token Storage](08-Token-Storage.md))

### Q2. Walk through the Authorization Code flow end to end.

#### Answer

1. Client redirects the user's browser to the authorization server's
   `/authorize` endpoint with `client_id`, `redirect_uri`, `scope`,
   `state`, and (for PKCE) `code_challenge`.
2. User authenticates with the authorization server and approves the
   requested scopes.
3. Authorization server redirects back to `redirect_uri` with a one-time
   `code` and the same `state` value — the client verifies `state` matches
   what it sent to rule out CSRF.
4. Client sends the `code` (plus `code_verifier` for PKCE, or the client
   secret for a confidential client) to the authorization server's
   `/token` endpoint.
5. Authorization server validates everything and returns an access token
   (and often a refresh token).
6. Client attaches the access token (typically `Authorization: Bearer
   <token>`) to requests against the resource server, which validates it
   before serving the request.

#### Follow-up Questions

- Why is the code exchanged in step 4 a *separate* server-to-server (or
  PKCE-verified) call instead of being returned directly in step 3?
- What stops an attacker who intercepts the redirect URL in step 3 from
  using the `code` themselves?

## Common Pitfalls

- Confusing OAuth (authorization — "can this app access this resource?")
  with authentication ("who is this user?") — plain OAuth doesn't
  guarantee anything about identity; that's what [OIDC](07-OIDC.md) is for.
- Still implementing the Implicit or Resource Owner Password Credentials
  flows for new apps instead of Authorization Code + PKCE.
- Skipping `state` validation, leaving the flow open to CSRF against the
  redirect callback.
- Treating the access token as something safe to store long-term in
  `localStorage` without considering XSS exposure (see
  [Token Storage](08-Token-Storage.md)).

## References

- [RFC 6749: The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- [RFC 7636: PKCE](https://datatracker.ietf.org/doc/html/rfc7636)
- [OAuth 2.0 Simplified](https://www.oauth.com/)
