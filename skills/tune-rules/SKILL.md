---
name: tune-rules
description: Author or adjust Dregs badge rules and escalation rules against real data. Read the docs, inspect existing rules, preview the impact, and create or change rules only after confirmation. Use when the user wants to label identities (badges), get notified or open incidents on a pattern (escalations), reduce false positives, or asks why a rule is or isn't firing.
---

# Tuning badge and escalation rules in Dregs

Badge rules label identities (Trusted, Suspicious, Bot, and so on) from score conditions. Escalation rules open
incidents when conditions hold, with a severity, optional notification channels, and open and close delays. Both are
team-wide and take effect on the next scoring pass, so a careless rule badges or pages the whole team.

## Before you write anything

1. **Read the docs.** Call `get_documentation` with `BADGES` before touching badge rules and `ESCALATIONS` before
   touching escalation rules, once per session. The chapters explain delays, severities, statuses, and how badge
   slugs are referenced.
2. **See what exists.** `list_badge_rules` and `list_escalation_rules`, then `get_badge_rule` or
   `get_escalation_rule` for anything you might change. Many requests are a tweak to an existing rule, not a new one,
   and duplicate rules are a common source of confusion later.
3. **Look at the data the rule will act on.** `dashboard_score_distribution` shows where scores actually sit, and
   `dashboard_rule_activity` shows which rules fire the most. A threshold that looks reasonable in the abstract can
   match half the customer base.
4. **Know the vocabulary.** The condition types and their arguments are in
   [references/conditions.md](references/conditions.md). All conditions in a rule must hold for it to match.

## Author, preview, confirm

1. **Draft the rule** and explain it to the user in plain terms: what it matches, what happens, and roughly how many
   identities it should affect.
2. **Preview escalation rules** with `preview_escalation_rule`, which counts the identities the conditions match
   today without saving anything. If the count surprises you or the user, adjust the thresholds or add an
   `IDENTITY_AGE_GT` condition before going further. Badge rules have no preview; use the score distribution and, if
   the rule keys off a category threshold, `list_identities` filtered to see who would be caught.
3. **Confirm, then create or update.** `create_badge_rule`, `update_badge_rule`, `create_escalation_rule`, and
   `update_escalation_rule` change the team's configuration and need the admin role. Wait for an explicit yes even
   if the client did not prompt.
4. **Channels.** Escalation rules deliver to channel slugs from `list_channels`. Channels themselves are created and
   edited only in the dashboard, and their secrets are never available to you. An empty channel list means
   dashboard-only, which is the right default for a new rule until it has proven itself.
5. **Check the effect.** After a scoring pass, or after `analyze_identity` on a few known examples (admin),
   `dashboard_rule_activity` shows what the rule caught. Offer to adjust.

## Reducing false positives

When a user says a rule is too noisy, look at the identities it caught with `list_identities` (by badge) or
`list_escalations` (by rule) and investigate a sample with the `investigate-identity` skill before changing anything.
The usual fixes are a tighter threshold, an added `IDENTITY_AGE_GT` condition so brand-new accounts settle first, an
`openAfterSeconds` delay, or a `BADGE_NOT_EXISTS` condition against a Trusted badge. Prefer adjusting the rule to
deleting it.

## Datasets and mappings

Some patterns are better handled by data than by thresholds. `list_datasets` and `list_dataset_entries` show the
lookup tables analyzers consult (disposable email domains, suspicious IPs, and the team's own lists), and
`add_dataset_entry` or `remove_dataset_entry` adjust the team's copies (admin). `list_mappings` and `set_mapping`
tell Dregs which of the customer's event names and identity fields mean registration, login, email, and so on, which
is often why an analyzer is silent. Confirm before every write, and never write to the GLOBAL scope on a customer's
behalf; that scope belongs to Dregs.

## Deleting

`delete_badge_rule` and `delete_escalation_rule` cannot be undone from the API. Deleting a badge rule breaks any
escalation rule that references its badge. Say so, confirm, and prefer disabling by editing the rule when the user
just wants quiet.
