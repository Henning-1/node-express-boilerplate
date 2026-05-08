# ADR 0001 — JWT access + refresh token strategy

**Status:** Accepted

## Context

We need short-lived access tokens for API authorization and a separate
mechanism for long-lived sessions without forcing re-login.

## Decision

- Access tokens: HS256-signed JWTs, default 30-minute expiry, configured
  via `JWT_ACCESS_EXPIRATION_MINUTES` in `src/config/config.js`.
- Refresh tokens: HS256-signed JWTs, 30-day expiry, stored server-side
  in the `tokens` collection so they can be revoked on logout / password
  change. Rotation handled in `src/services/token.service.js`.

## Consequences

- Compromised access tokens are valid for at most 30 minutes.
- Refresh tokens leak ⇒ explicit revocation needed.
- **Anti-pattern:** setting `JWT_ACCESS_EXPIRATION_MINUTES` to 0 or
  negative breaks all token flows. Validate in CI before merge to main.