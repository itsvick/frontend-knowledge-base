# OIDC

> Part of: 13-Security

## Overview

OpenID Connect (OIDC) is an **authentication** layer built directly on top
of [OAuth 2.0](06-OAuth.md). Where OAuth only answers "does this app have
permission to access this resource?", OIDC adds a standardized way for the
client to also learn "who is this user?" — via a signed **ID token** and a
standard `/userinfo` endpoint. It's what "Sign in with Google/Microsoft/
Apple" is built on.

## Key Concepts

- **ID token** — a [JWT](05-JWT.md), always signed, containing identity
  claims about the authenticated user: `sub` (stable unique user id),
  `iss` (issuer), `aud` (client id it was issued for), `exp`/`iat`, plus
  optional profile claims (`email`, `name`, `picture`) if requested via
  scope. It's meant to be *read* by the client to establish identity, not
  sent to a resource server as a bearer credential — that's the access
  token's job.
- **Access token vs ID token** — both are often JWTs and both come back
  from the same `/token` exchange, but they serve different purposes: the
  **access token** is opaque-to-the-client and presented to a resource
  server to authorize API calls; the **ID token** is meant for the client
  itself to parse and verify, establishing who just logged in. Sending an
  ID token to a resource server as if it were an access token is a common
  misuse — resource servers shouldn't accept it.
- **`openid` scope** — requesting the `openid` scope in an OAuth
  authorization request is what turns a plain OAuth flow into an OIDC flow
  and triggers issuance of an ID token alongside the access token.
- **`/userinfo` endpoint** — a standard OIDC endpoint the client can call
  with the access token to fetch additional profile claims about the user,
  useful when you don't want to bloat the ID token itself.
- **Discovery document** (`/.well-known/openid-configuration`) — a
  standardized JSON document every OIDC provider exposes, listing its
  `authorization_endpoint`, `token_endpoint`, `jwks_uri` (public keys for
  verifying ID token signatures), supported scopes/claims, etc. — lets
  client libraries auto-configure against any compliant provider.
  `jwks_uri` specifically is what lets a client verify an ID token's
  signature without hardcoding the provider's public key.
- **Nonce** — an OIDC-specific parameter (distinct from OAuth's `state`),
  sent in the authorization request and echoed back inside the ID token's
  claims, binding the specific ID token to the specific browser session
  that requested it and preventing replay of a stolen/reused ID token from
  a separate login.

## Interview Questions & Answers

### Q1. What's the actual difference between OAuth and OIDC?

#### Answer

OAuth is purely about **authorization** — it issues an access token that
proves "this client has permission to call this API with these scopes,"
with no standardized concept of who the user is. OIDC is a thin identity
layer added on top of OAuth's Authorization Code flow: requesting the
`openid` scope makes the authorization server also return a signed **ID
token**, a standardized JWT that tells the client exactly who
authenticated (`sub`, plus optional profile claims), which is what
"authentication"/"login" actually requires. In short: OAuth answers "can
this app do X," OIDC answers "who is this user," and OIDC is layered on
OAuth's existing flows rather than being a separate protocol.

#### Follow-up Questions

- Why can't you safely use a plain OAuth access token as proof of a user's
  identity?
- What would go wrong if a resource server accepted an ID token in place
  of an access token?

### Q2. How does a client verify an ID token is legitimate?

#### Answer

- Verify the signature using the provider's public key, fetched from
  `jwks_uri` (found via the provider's discovery document) — never trust
  an unverified token.
- Check `iss` matches the expected authorization server.
- Check `aud` matches this client's own `client_id` (so a token issued for
  a *different* app can't be replayed against this one).
- Check `exp`/`iat` for validity window.
- Check `nonce` matches the value this client generated for this specific
  login attempt, preventing replay of a previously issued ID token.

#### Code Example

```js
// Using a library like `openid-client` (illustrative, not exhaustive)
const { access_token, id_token } = await client.callback(redirectUri, params, {
  nonce: storedNonce,
  state: storedState,
});

// The library verifies signature, iss, aud, exp, and nonce internally
const claims = jwt.decode(id_token); // { sub, iss, aud, email, ... }
```

#### Follow-up Questions

- Why is checking `aud` important even though the signature is already
  valid?
- If your ID token verification skipped the `nonce` check, what attack
  becomes possible?

## Common Pitfalls

- Using the ID token as if it were an access token to call a resource
  server's API — resource servers should only accept access tokens they
  themselves are the intended audience for.
- Trusting an ID token's claims without verifying `iss`/`aud`/signature,
  effectively accepting any correctly-formatted token.
- Confusing OAuth's `state` (CSRF protection for the redirect) with OIDC's
  `nonce` (replay protection for the ID token) — they solve different
  problems and both should be used together.
- Assuming "OAuth login" implies real authentication when the app never
  requested the `openid` scope or verified an ID token — that's just
  authorization, with no verified identity behind it.

## References

- [OpenID Connect Core 1.0 spec](https://openid.net/specs/openid-connect-core-1_0.html)
- [OpenID Connect explained (Auth0)](https://auth0.com/docs/authenticate/protocols/openid-connect-protocol)
- [OAuth 2.0 and OpenID Connect (in plain English) — YouTube, OktaDev](https://www.youtube.com/watch?v=996OiexHze0)
