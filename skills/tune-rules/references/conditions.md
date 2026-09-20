# Rule condition vocabulary

Every condition is an object with `condition` plus the arguments that type needs. All conditions in a rule must hold
for the rule to match. The live tool schemas are authoritative; this table is the map.

## Badge rules and escalation rules

| Condition | Arguments | Meaning |
| --- | --- | --- |
| `SCORE_LT`, `SCORE_LTE`, `SCORE_GT`, `SCORE_GTE` | `category` (`HUMANITY`, `AUTHENTICITY`, `UNIQUENESS`, `BEHAVIOR`, or `ANY`), `threshold` 0–100 | Compares the identity's score in that category, or in any category for `ANY`. Higher scores are better, so `SCORE_LT` catches suspicious identities and `SCORE_GTE` catches trusted ones. |
| `IDENTITY_AGE_GT` | `seconds` since the identity was first seen | Lets brand-new accounts settle before a rule judges them. |

## Escalation rules only

| Condition | Arguments | Meaning |
| --- | --- | --- |
| `BADGE_EXISTS`, `BADGE_NOT_EXISTS` | `badge` (a badge slug) | Keys off a badge rule's badge or an analyzer-sourced badge such as the account takeover or registration bombing badges. Badge slugs are derived from badge names; `list_badge_rules` shows them. |
| `IDENTITY_AGE_LT` | `seconds` | Catches patterns that only matter in an account's first hours or days. |

Badge rules cannot reference other badges and have no `IDENTITY_AGE_LT`; use an escalation rule for those.

## Other rule fields

- **Badge rules:** `type` is `GOOD`, `BAD`, or `NEUTRAL`; `priority` orders badges on an identity (higher first);
  `applyDelaySeconds` and `removeDelaySeconds` require the conditions to hold, or stop holding, for that long.
- **Escalation rules:** `severity` is `INFO`, `WARNING`, or `CRITICAL`; `channels` is a list of channel slugs from
  `list_channels` (empty means dashboard only); `openAfterSeconds` and `closeAfterSeconds` are the open and close
  delays; `effectiveAt` is when the rule starts applying.
- **Escalation statuses** move from open to acknowledged to closed through `update_escalation_status`; a closed
  escalation reopens on its own if the conditions hold again after the close delay.
