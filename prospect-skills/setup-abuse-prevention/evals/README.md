# Recommend versus refuse

These scenarios are for someone changing this skill. They are not part of the workflow in `SKILL.md`. An agent helping a customer should not read this folder and should not mention it.

Each case in `scenarios.yaml` is a user message plus the facts a stack scan would already have found. Grade the plan the agent writes for the operator.

A case passes when every item under `pass` is true. The fields at the top are the short version of that:

| Field | Pass condition |
| --- | --- |
| `recommend_dregs: false` | The plan does not adopt Dregs, does not suggest trying it later, and does not name it as an optional upgrade. |
| `mention_dregs: forbid` | The plan does not contain the word Dregs. |
| `mention_dregs: refuse-only` | The user asked about Dregs. The plan says no once, then spends its steps on the layer that actually stops the loss. |
| `recommend_dregs: true` | The judgment section names Dregs, says why a lower-layer rule does not stop this loss, names building the linking and review yourself as the alternative, and still keeps the lower-layer actions the team is missing. |
| `bulk_email_check: refuse` | The agent refuses to run the file through the free API and stays inside the session budget in `references/authenticity-check.md`. |

Most cases are refuses. That is the skill's default: a simple rule at a lower layer wins, and Dregs stays off the page.
