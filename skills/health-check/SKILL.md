---
name: health-check
description: Diagnose the quality of a Dregs integration, read-only. Checks that events arrive, users are identified, event names and identity fields are mapped, analyzers produce observations, and the account is within its limits, then says what to fix. Use when scores look wrong or all sit in the middle, identities are anonymous, analyzers seem silent, or the user asks whether their Dregs setup is working.
---

# Checking the health of a Dregs integration

Most "Dregs isn't working" reports are one of five things: no events, events with no identity, unmapped event names,
unmapped identity fields, or an exceeded event limit. This skill finds which, without changing anything.

## Checks

1. **Account.** `get_account_summary`: the team, the plan, and usage against limits. An exceeded monthly event limit
   pauses ingestion, so if it is exceeded, stop here and report that first; every other symptom follows from it.
2. **Ingestion.** `dashboard_stats` for the last seven days: events, active identities, scored identities. Zero or
   near-zero events means the tracking snippet or backend client is not sending; check the `setup-tracking` and
   `setup-events` skills' verification steps.
3. **Identification.** `list_identities` sorted by recent activity. Identities without a display name or email, or
   with IDs that look like session tokens, mean `dregs.identify()` is not called, is called with an unstable ID, or
   runs before the script initializes. Compare the share of anonymous identities to the total.
4. **Identity stitching.** For a couple of identified users, `search_events` filtered to the identity: do frontend
   and backend events appear under one identity? Split identities mean the frontend and backend use different IDs.
5. **Mappings.** `list_mappings` for kind `EVENT_TYPE` and kind `IDENTITY_FIELD`, scope `CUSTOMER`, against the
   event names seen in `search_events` and the attribute keys seen in `get_identity`. Registration, login, email,
   and name are the mappings that matter most; missing ones explain quiet analyzers and middling authenticity
   scores.
6. **Observations.** `get_identity_analysis` on three or four recently scored identities. Count how many analyzers
   produced observations and read the ones that say they lacked data. Few observations with high confidence means
   the data is fine and the users are just ordinary; few observations overall means analyzers cannot see the
   signals they need.
7. **Distribution.** `dashboard_score_distribution`: scores piled in the middle across categories usually means
   low confidence from thin data; a healthy account is mostly high with a low tail.
8. **Rules and deliveries.** `list_badge_rules`, `list_escalation_rules`, and `escalation_summary` to see whether
   anything is acting on the scores, and `list_channels` plus `list_channel_deliveries` for failed deliveries.

## Report

Lead with the verdict: working, partly working, or not receiving data, and the single most important fix. Then a
short table of the checks with pass, warn, or fail and one line of evidence each. Then the fixes in order, each
pointing at the skill that does it (`setup-tracking`, `setup-events`, `setup-webhooks`, `tune-rules`).

Do not change anything during a health check. If the user wants a fix applied, switch to the relevant skill and
confirm each change there.
