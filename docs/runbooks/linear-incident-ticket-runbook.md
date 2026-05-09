# Runbook: Open an incident Linear tracking ticket

Every declared production incident gets exactly one Linear ticket,
opened via `create_linear_issue`. The ticket is the post-incident
review's single anchor — every artifact (Slack thread, GitHub issue,
PR, commit, runbook, ADR) hangs off it.

## Title

One line, present tense, names the system + the bug class:

- "Auth 401s on prod — JWT_ACCESS_EXPIRATION_MINUTES regression"
- "Checkout 500s — order serializer canary gap"
- "TechDocs build failing — generator OOM"

## Defaults

- `priority`: 2 (high) for sev-2; bump to 1 for sev-1.
- `state`: "In Progress" at creation. Move to "Done" only after the
  fix is shipped *and* the post-incident review is filed.
- `team_key`: CLI (cline-hackathon).

## Description (markdown) — required sections

The description must be markdown and must include all seven sections
below, in this order:

1. **Diagnosis.** One paragraph: symptom + most likely root cause +
   confidence level. State why ("Confidence: high because <2-3
   reasons>").
2. **The breaking change.** PR number (linked), commit SHA, the
   exact `file:line` that broke, and the commit author surfaced by
   `git_blame`.
3. **Framing vs effect.** One line calling out any gap between the
   PR's stated description (title + body) and what the diff
   actually did. If the PR was honest about its effect, say so
   explicitly — silence here is ambiguous.
4. **Owner.** The CODEOWNERS owner of the broken file, surfaced via
   `who_owns`. This is the person to page.
5. **Citations.** Links to the runbook section + ADR clause this
   change violates. Quote the relevant clause text inline so the
   ticket is self-contained.
6. **Mitigation.** A concrete revert command using the commit SHA
   (e.g. `git revert <sha>`), plus the expected ETA.
7. **References.** Links to the Slack incident channel, the GitHub
   incident issue, and the canonical surface runbook.

## After creating the ticket

Post the Linear URL into the incident Slack channel as a follow-up
message in the format:

  `Tracking: <linear-url|CLI-N>`

This closes the loop — anyone watching the channel can jump straight
into the tracking ticket.

## Owners

Runbook author: `@alice-platform`. Edits require their review.
