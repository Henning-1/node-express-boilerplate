 # Runbook: Auth 401-storm                                                                                                                  14:19:18 [35/1875]

  Reference for triaging "auth is throwing 401s" incidents in this service.
  Updated after every auth-related sev2.

  ## 1. Symptoms

  - Wall of 401s on `/v1/auth/*` and on any route guarded by
    `src/middlewares/auth.js`.
  - Datadog `auth-prod` dashboard goes red, often within minutes of a deploy.
  - Sentry surfaces JWT-related errors: `TokenExpiredError`,
    `JsonWebTokenError`, `NotBeforeError`.

  ## 2. Triage checklist

  Work through these in order — most auth failures fall into one of these
  buckets. Use `git log --oneline main..main~20` to identify recent
  merges that could have changed any of these surfaces.

  1. **Issuer health.** `curl -fsS $API/v1/health` should 200. If the
     issuer pod is down or crashlooping, no fresh tokens are minted and
     the 401s are a downstream symptom.

  2. **Recent deploys.** A 401-storm that starts within minutes of a
     deploy is almost always a config or middleware regression. Check
     the merged PR list since the last green window.

  3. **Token lifetime configuration.** Token TTLs live in
     `src/config/config.js` (`JWT_ACCESS_EXPIRATION_MINUTES`,
     `JWT_REFRESH_EXPIRATION_DAYS`). Misconfigured values here cause
     tokens to be rejected before they're used. ADR-0001 documents the
     constraints these values must satisfy.

  4. **Secret rotation.** A mismatched `JWT_SECRET` between the issuer
     pod and the verifier pod produces `JsonWebTokenError: invalid
     signature` on every request. Roll all pods to the same secret
     before investigating anything else.

  5. **Refresh path.** `src/services/token.service.js` `verifyToken()`
     and `refreshAuth()` in `src/services/auth.service.js` reject on
     blacklisted or rotated tokens. A stale or runaway `tokens`
     collection can surface as 401s on the refresh endpoint
     specifically — check before blaming the access token.

  6. **Middleware.** `src/middlewares/auth.js` is the single ingress
     point for token verification — any `passport-jwt` strategy change
     or a bad require-path here can 401 the entire app.

  ## 3. Mitigation patterns

  - **Config regression:** revert the offending PR. Validate resulting
    values against ADR-0001 before merging the revert.
  - **Secret mismatch:** roll all pods to the same `JWT_SECRET`, force a
    client-side refresh wave.
  - **Blacklist leak:** truncate `tokens` rows older than the access TTL
    and restart the issuer pod.

  ## 4. Communication template

  sev2 — auth 401s on prod after deploy.
  Suspected cause: <section #N from this runbook>.
  Owner: @alice-platform (CODEOWNERS).
  Mitigation: <revert PR # / roll pods / truncate>.
  ETA to mitigation: <5 min>.

  ## 5. Owners

  - Runbook author: @alice-platform
  - Auth files (`src/middlewares/auth.js`, `src/services/token.service.js`,
    `src/services/auth.service.js`, `src/config/config.js`,
    `src/routes/v1/auth.route.js`): @alice-platform per CODEOWNERS.

  ## 6. Related references

  - ADR-0001 — JWT access + refresh token strategy (constraints on the
    expiry values in `src/config/config.js`).
  - ADR-0002 — refresh token rotation on use.