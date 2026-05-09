On-call here. Active incident:

Walk through this in order — do NOT skip steps:

──────────────────────────────────────────────────────────────────
STEP 1 — Diagnose
──────────────────────────────────────────────────────────────────
Call diagnose_incident with the symptom above.
KEEP a record of EVERY URL the tool returned across:
  • similar_incidents[*].url      (Slack threads + Linear tickets)
  • runbook[*].url                (GitHub runbook lines)
  • owners (will appear in the description as text, no URL)
You will need every single one in Step 2.

Optionally call get_runbook + find_similar_incidents with related
queries if diagnose_incident's hits don't fully cover the symptom —
add any new URLs to your running list.

──────────────────────────────────────────────────────────────────
STEP 2 — Open the Linear ticket WITH FULL SOURCE TRAIL
──────────────────────────────────────────────────────────────────
Call create_linear_issue with:

  title:    "Auth 401s on prod — incident-2026-05-09 — JWT expiry regression"
  priority: 2
  state:    "In Progress"
  description: a markdown body with EXACTLY these sections, in this order:

    ## Diagnosis
    4–7 sentences. Be specific — generic summaries fail this
    section. Required content, in any order:
      - What's broken: name the specific endpoints, error rate, and
        scope (% of traffic / users affected).
      - Root cause: name the specific config knob, function, or
        code path. If a value changed, write `<old> → <new>`.
      - Mechanism: 1–2 sentences explaining *why* this change
        produces the symptom (the causal chain).
      - Confidence: explicit level (high / medium / low) followed
        by 2–3 specific reasons, each citing a source — past
        incident link, runbook §N, or blame trail.

    Bad example: "Auth is failing because of a recent change to
    JWT config. Confidence: high."

    Good example: "401-storm on `/v1/auth/*` and every route
    guarded by the auth middleware (~100% of authenticated
    traffic). Root cause: `JWT_ACCESS_EXPIRATION_MINUTES` in
    `src/config/config.js` went `30 → 0` in PR #2 (merged 22 min
    ago). With a 0-minute TTL, every freshly-minted token is
    already expired by the time the client sends it back, so
    `verifyToken()` rejects on `TokenExpiredError`. Confidence:
    high — (1) symptom matches past incident
    `#incident-2025-03-auth-down` (CLI-5), (2) auth runbook §3
    explicitly names this knob as the first thing to check after a
    deploy-correlated 401-storm, (3) `git_blame` on the suspect
    line lands on PR #2's commit."

    ## Suspect change
    - **PR:** <title> (<linked PR url>) by <author>, merged
      <relative time>; reviewer (if any from get_pr_diff)
    - **File:** `<path>:<line range>` — quote the changed line(s)
      inline as a fenced code block, before-and-after:
      ```
      - <old line>
      + <new line>
      ```
    - **Commit SHA:** `<sha>`
    - **Author:** <git_blame author> (may differ from PR author
      for rebased / squashed commits)
    - **Framing vs effect:** quote the PR's stated description
      verbatim as a blockquote (1–3 lines), then one line on the
      gap vs the actual diff. If the description matched the diff,
      state that explicitly: "PR description matches the diff." —
      silence is ambiguous.

    ## Impact
    - **Surface:** which user-facing flows / endpoints / jobs
      broke.
    - **Volume:** error rate or % of traffic affected; cite the
      source (Datadog dashboard URL, Sentry, log query).
    - **Duration:** start time relative to deploy + how long it's
      been running.
    - **Data integrity:** "no — availability only", or specifics
      if there's data risk.

    ## Owner
    - **Primary:** `<github handle>` per CODEOWNERS rule
      `<pattern>` — this is the person to page.
    - **Backup:** if CODEOWNERS lists a fallback or wildcard rule
      that also matches, name it. Otherwise: `_(none)_`.
    - **Slack member ID:** the U… ID for the primary owner so the
      on-call coordinator can DM them directly.

    ## Mitigation
    - **Revert command:** literal `git revert <sha>` (or a specific
      config-rollback command if the change wasn't a single
      commit).
    - **Validation step:** the concrete command or dashboard URL
      that proves the fix landed (e.g.
      `curl -s $API/v1/auth/login` returns 200, or "Datadog
      `auth-prod` dashboard returns to green").
    - **ETA:** integer minutes.
    - **Constraint quote:** quote inline the runbook clause or
      ADR clause this change violated, e.g. "ADR-0001:
      'access-token TTL MUST be >= 1 minute'".

    ## Sources
    Every URL the diagnosis was grounded in. Use markdown
    [label](url) format. Group as:

    **Slack threads**
    - [<channel> · <one-line excerpt>](url)
    - … (one bullet per Slack url)

    **Linear (past incidents)**
    - [<identifier> — <title>](url)
    - … (one bullet per Linear url)

    **GitHub (runbooks · ADRs · code)**
    - [<file_path> §<section>](url)
    - … (one bullet per GitHub url, including line ranges)

    ## Generated by
    `ctx-mcp · diagnose_incident` · <utc timestamp>

If a section has zero entries, write "_(none)_" — do not omit the
heading. Do not summarize away URLs; cite each one verbatim.

Note the returned ticket identifier (CLI-N) and url.

──────────────────────────────────────────────────────────────────
STEP 3 — Open the incident Slack channel with the synthesis
──────────────────────────────────────────────────────────────────
Call create_slack_channel to stand up a dedicated coordination
channel for this incident. Pass:

  name:    incident-YYYY-MM-DD-<system>-<symptom-shorthand>
           (e.g. "incident-2026-05-09-auth-401s") — use the
           date the incident *started*, lowercase, hyphens only.
  topic:   one-line present-tense incident description
           (e.g. "Active sev-2 — auth 401s on prod after deploy").
  purpose: "Coordination channel for the [system] [symptom]
           incident. Updates here are the source of truth until
           resolution."
  invite:  ALWAYS include both of the following Slack member IDs
           verbatim — these are the standing incident-manager
           rotation and must be in every incident channel:
             - U0B1SMKB74N  (Chetan Singh)
             - U0B1VBP0B7U  (Henning)
           Plus the CODEOWNERS owner of the broken file identified
           in Step 2 (look it up via who_owns if not already in
           your context). If who_owns returns a github handle (e.g.
           "@alice-platform"), translate to the Slack member ID
           via the slack_user index — pass the U… ID, not the
           handle.

           Pass these as a list of strings to the `invite` param,
           e.g. invite=["U0B1SMKB74N", "U0B1VBP0B7U", "<owner-id>"].
           The Slack API rejects display names — only U-prefixed
           member IDs are accepted. Duplicates are fine; invites
           are idempotent.
  initial_message: the Slack mrkdwn synthesis below — use *bold*,
                   _italic_, <url|label>:

    *Diagnosis* — 2 sentences. Name the specific thing that broke
    (config knob, function, route) and the change that caused it
    (old value → new value, or "added/removed X"). Don't write
    "auth is failing" — write "JWT_ACCESS_EXPIRATION_MINUTES went
    30 → 0 in src/config/config.js, so every freshly-minted token
    expires instantly".
    *Confidence:* high — <name 2-3 specific reasons: matched past
    incident X (linked), runbook §N points here, blame trail lands
    on Y>.

    *Suspect change*
    • <PR linked> by <author>, merged <relative time>
    • `<file>:<line>` — was `<old>` → now `<new>`
    • Commit `<sha>`
    • Framing: <quote the PR's stated description verbatim, then
      one line on the gap vs the actual diff>

    *Owner to page:* <slack @-mention or U… ID> · CODEOWNERS rule
    `<pattern>`

    *Mitigation*
    • Revert: `git revert <sha>`  (or `<specific config rollback>`)
    • Verify: <concrete command or dashboard URL that proves the
      fix landed>
    • ETA: <integer> min

    *Tracking:* <CLI-N linked>

    *Sources:* 2–3 most relevant URLs as <url|label> — full trail
    in the Linear ticket.

    _:robot_face: ctx-mcp synthesis_

Note the returned channel URL — you'll need it for Step 4.

──────────────────────────────────────────────────────────────────
STEP 4 — Reply
──────────────────────────────────────────────────────────────────
Reply here with three lines only:
  • Linear: <CLI-N> <url>
  • Slack:  <channel url>
  • Sources cited: <count>