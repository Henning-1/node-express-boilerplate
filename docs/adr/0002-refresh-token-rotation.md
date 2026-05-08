# ADR 0002 — Refresh token rotation on use

**Status:** Accepted

## Context

A refresh token reused after the client received a fresh one is a
likely indicator of theft.

## Decision

`refreshAuth()` in `src/services/auth.service.js` deletes the prior
refresh token row before issuing the new pair. If the deletion fails,
abort the rotation — do not return new tokens.

## Consequences

- Replay attacks against the refresh endpoint are detectable.
- Race condition risk when the same refresh token is sent twice in
  parallel (e.g., aggressive client retries) — second call hits a
  missing row and 401s. Acceptable.