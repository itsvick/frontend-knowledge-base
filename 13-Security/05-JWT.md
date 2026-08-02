# JWT

> Part of: 13-Security

## Overview

A JSON Web Token (JWT) is a compact, URL-safe, self-contained token format
for representing claims (e.g. "this is user 123, issued at T, expires at
T+1h") that can be verified without a database lookup, because it's signed
(or encrypted) by the issuer. It's widely used as the bearer token in
OAuth/OIDC flows and for stateless session auth in APIs. See also
[OAuth](06-OAuth.md), [OIDC](07-OIDC.md), and
[Token Storage](08-Token-Storage.md) for how JWTs fit into a full auth flow.

## Key Concepts

- **Structure: `header.payload.signature`** — three base64url-encoded
  segments joined by dots. The header declares the algorithm (`alg`) and
  token type; the payload holds claims; the signature is computed over
  `header + '.' + payload` using the algorithm/key from the header, so any
  tampering with header or payload invalidates it.
- **Claims** — key/value pairs in the payload. Registered/standard claims
  include `iss` (issuer), `sub` (subject/user id), `aud` (audience), `exp`
  (expiry), `iat` (issued at), `nbf` (not before); apps add custom claims
  (roles, permissions) on top.
- **Signing vs encryption** — a standard JWT (JWS) is *signed*, not
  encrypted: the payload is only base64-encoded, so it's readable by
  anyone who has the token, just not forgeable without the signing key.
  If the payload must be confidential, use JWE (encrypted JWT) or don't put
  sensitive data in the claims at all.
- **`HS256` vs `RS256`** — `HS256` uses one shared secret to both sign and
  verify (symmetric); every service that verifies tokens must hold that
  same secret, so a leak anywhere lets that party forge tokens. `RS256`
  uses an asymmetric key pair: the issuer signs with a private key, and any
  number of services can verify with the corresponding public key without
  being able to forge tokens themselves — the right choice when
  verification happens in multiple services/servers.
- **Statelessness trade-off** — because a valid signature is equivalent
  to "trust this," the server doesn't need a session store to validate a
  request, which is great for scaling. The cost is **revocation**: a JWT
  issued with `exp` 1 hour out remains valid for that hour even if the
  user is banned or logs out, unless you add an extra layer (short expiry +
  refresh tokens, or a server-side revocation/deny-list check, which
  reintroduces state).
- **`alg: none` / algorithm confusion attacks** — a classic JWT
  vulnerability where a library accepts a token whose header claims
  `alg: none` (no signature required) or where an `RS256`-issued token is
  re-verified as `HS256` using the public key as the HMAC secret. Mitigated
  by explicitly pinning the expected algorithm(s) when verifying, never
  trusting the `alg` field from the token itself.

## Interview Questions & Answers

### Q1. What is a JWT, and how is it different from an opaque session token?

#### Answer

A JWT is self-contained and self-verifying: the payload (claims) and a
signature travel together, so any server holding the verification key
(shared secret or public key) can validate it and read its claims without
querying a database or session store. An opaque session token (e.g. a
random UUID in a cookie) carries no information by itself — the server
must look it up in a session store to know who it belongs to and whether
it's still valid.

The trade-off: JWTs scale better (no shared session store, easy to verify
in multiple services) but are hard to revoke early since validity is
determined purely by the signature and `exp`, not a live check. Opaque
tokens are trivial to revoke (delete the session row) but require a lookup
on every request.

#### Follow-up Questions

- If you need to support "log out everywhere" immediately, what would you
  add to a pure stateless-JWT design?
- Why is it unsafe to store sensitive data (e.g. password hash, full PII)
  in a JWT payload?

### Q2. How do you verify a JWT securely, and what's the "alg: none" attack?

#### Answer

Verification means recomputing the signature over the header+payload using
the algorithm and key **you expect** (configured server-side), and
comparing it to the token's signature — never trusting the `alg` field
embedded in the token itself, since a malicious client controls that
field. The `alg: none` attack exploits libraries that honor an attacker-set
`alg: none` header and skip signature verification entirely, effectively
accepting any payload as valid. A related attack sends an `RS256`-issued
token but sets `alg: HS256` in the header; a naive verifier that picks the
algorithm from the token, then uses the (public, widely known) RSA public
key as the HMAC secret, ends up "verifying" a token the attacker forged
themselves with that public key.

Mitigation: always pin the accepted algorithm(s) explicitly in the
verification call (most JWT libraries accept an `algorithms: ['RS256']`
option) instead of trusting the token's own header.

#### Code Example

```js
import jwt from 'jsonwebtoken';

// Signing (issuer, holds the private/shared key)
const token = jwt.sign({ sub: 'user-123', role: 'admin' }, privateKey, {
  algorithm: 'RS256',
  expiresIn: '15m',
});

// Verifying (any service with the public key) — pin the algorithm explicitly
const payload = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

#### Follow-up Questions

- Why should access tokens generally have a short expiry (minutes) rather
  than hours/days?
- How would you rotate the signing key without invalidating tokens issued
  under the old key mid-flight?

## Common Pitfalls

- Storing a JWT in `localStorage` to make it "easy to attach to requests,"
  which exposes it to any XSS bug on the page (see
  [Token Storage](08-Token-Storage.md) for the trade-offs).
- Treating a JWT's contents as trusted/confidential — it's signed, not
  encrypted, so anyone who intercepts it can read the payload.
- Giving access tokens long expiries "so users don't get logged out," which
  makes a leaked/stolen token dangerous for a long window since it can't be
  revoked early without extra infrastructure.
- Not validating `exp`, `nbf`, `iss`, and `aud` — accepting any
  correctly-signed token regardless of who it was issued for or when.

## References

- [jwt.io — JWT Debugger & Introduction](https://jwt.io/introduction)
- [RFC 7519: JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)
- [Auth0: JWT Handbook](https://auth0.com/resources/ebooks/jwt-handbook)
