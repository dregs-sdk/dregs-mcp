---
name: weekly-review
description: Produce a periodic health review of a Dregs account. Summarize scoring activity, the score distribution, which rules and badges fired, open escalations, and plan usage, then recommend what to look at or tune. Use when the user asks for a weekly or monthly review, a summary of what Dregs caught, a fraud report, or what needs attention.
---

# Reviewing a Dregs account

A review answers four questions: what came in, how it scored, what the rules did about it, and what is still waiting
for a human. Read first, write nothing without asking.

## Gather

1. **`get_account_summary`** for the team, the plan, and usage against limits. Call out an approaching or exceeded
   event limit first, because an exceeded limit pauses ingestion and everything below it is stale.
2. **`dashboard_stats`** for the window (default the last seven days; pass `from` and `to` for anything else): new
   and active identities, events, and how many identities were scored.
3. **`dashboard_score_distribution`** for the same window. Describe the shape per category: a healthy account is
   mostly high with a thin low tail; a bimodal distribution means a distinct population of bad accounts; a broad
   middle usually means analyzers lack the events or mappings they need to be confident.
4. **`dashboard_rule_activity`** for which badge rules and escalation rules fired and on whom. The top few identities
   in each group are worth a glance with `get_identity`; a rule that fired on nothing may be misconfigured, and one
   that fired on everything is noise.
5. **`escalation_summary`** and `list_escalations` filtered to open, then acknowledged. Note age and severity. If
   channels are in use, `list_channel_deliveries` for any channel with failures.

## Report

Write for someone who has not opened the dashboard this week:

- Volume and change: identities and events versus the previous period if the user gave one.
- Score health: the distribution in one or two sentences per category, with the notable movers.
- What the rules caught: badges applied and escalations opened, grouped by rule, with two or three example
  identities and their `dashboardUrl` links.
- What is waiting: open and acknowledged escalations by severity, oldest first.
- Recommendations: a short list, each tied to evidence above. Typical items are a threshold to tighten or loosen, a
  mapping to add because an analyzer is silent, a stale escalation to close, or an identity cluster worth an
  investigation with the `investigate-identity` skill.

Keep the register plain. Numbers go in a table, verdicts go in sentences, and anything you are not sure about is
flagged as such.

## Act only on request

Everything above is read-only. If the user wants to act on a recommendation, use `update_escalation_status` for
triage, and the `tune-rules` skill for rule changes, each with its own confirmation. Do not close escalations in bulk
to make the report look clean.
