# Runbook: Open an incident Slack channel

When a production incident is declared, the on-call engineer (or an
agent acting on their behalf) opens a dedicated coordination channel
using the `create_slack_channel` tool. This runbook is the source of
truth for naming, membership, and the opening message.

## Naming

- Format: `incident-YYYY-MM-<system>-<symptom-shorthand>`
- Examples:
  - `incident-2026-05-auth-down`
  - `incident-2025-09-checkout-500s`
  - `incident-2025-03-db-failover`
- Use the date the incident *started*, not the date it was reported.
- `<system>` is the affected surface (auth, checkout, db, techdocs).
- `<symptom-shorthand>` is 1–3 hyphenated words.

## Topic

A single line, present tense, no jargon:

  "Active sev-2 — auth 401s on prod after deploy"

## Purpose

A two-sentence purpose, set on the channel:

  "Coordination channel for the [system] [symptom] incident.
  Updates here are the source of truth until resolution."

## Members to invite

Always invite the standing **incident-manager rotation**:

- `@chetan` — Incident manager (rotation)
- `@henning` — Incident manager (rotation)

Plus the **surface owner** from CODEOWNERS:

- For auth-surface incidents (anything under
  `src/middlewares/auth.js`, `src/services/token.service.js`,
  `src/services/auth.service.js`, `src/config/config.js`,
  `src/routes/v1/auth.route.js`):
  - `@alice-platform` — Platform/auth lead, runbook author
- For services + controllers (anything under `src/services/` or
  `src/controllers/`):
  - `@dan-coretech` — Core tech lead
- If unsure, call `who_owns` on the file most likely involved and
  invite whoever it returns.

Pass the union of these handles to the `invite` parameter of
`create_slack_channel`. Duplicates are fine — invites are idempotent.

## Opening message

Post a single message at channel creation (`initial_message` param)
that gives anyone joining the channel immediate context:

- One line: symptom + start time relative to the last deploy.
- Suspected deploy window (if known).
- Link to the canonical runbook for the affected surface
  (e.g. `docs/runbooks/auth-runbook.md`).
- Link to the GitHub incident issue (if one exists).
- A line: "Tracking ticket coming in Linear."

Format the message as Slack mrkdwn (`*bold*`, `_italic_`,
`<url|label>`).

## Owners

Runbook author: `@alice-platform`. Edits require their review.
