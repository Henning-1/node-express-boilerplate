# Runbook: Postgres failover warmup

Used when the primary Postgres instance is failed over to a hot standby and
the application surfaces high p99 latency for 1–3 minutes.

## Symptoms

- p99 spikes to 1–5 seconds for `/v1/users`, `/v1/auth/refresh-tokens`,
  any read-heavy route.
- Datadog `db-prod` dashboard shows the new primary at 100% CPU on
  `BufferAlloc`-style waits.

## Mitigation

The new primary's shared buffer cache is cold. Run:

```sh
psql -c "SELECT pg_prewarm('users');"
psql -c "SELECT pg_prewarm('tokens');"
psql -c "SELECT pg_prewarm('idx_users_email');"
```

p99 returns to baseline within 60–90 seconds of `pg_prewarm` completing
on the hottest tables. Document the recovery in the incident channel
with the prewarm timing for the next on-call.