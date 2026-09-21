---
name: setup-webhooks
description: Receive Dregs webhooks in the user's application, verify their signatures, and act on scores and escalations. Use when the user wants to build an endpoint for Dregs webhooks, verify the HMAC signature, react to score changes or escalations in their own system, or asks how to automate a response to fraud signals.
---

# Setting up webhooks

Webhooks turn Dregs from a dashboard into a response system: when an identity is scored, its badges change, or an
escalation opens or closes, Dregs POSTs to the user's endpoint, and their code decides what to do. This skill builds
the receiver and the first response; the channel itself is created in the dashboard.

## Before you write anything

1. Call `get_documentation` with `WEBHOOKS` and `CHANNELS`. The payload shape, the three request headers, the
   HMAC-SHA256 verification steps, the retry schedule, and the event types are defined there. Use them; do not
   reconstruct from memory.
2. Decide the response with the user before writing code. Typical first responses, in increasing force:
   record the scores on the user record for later queries; flag the account for manual review when a score
   crosses a threshold; hold a risky action (publishing, inviting, payouts) behind extra verification; quarantine or
   suspend on a critical escalation. Start with recording and flagging. Suspending users from a webhook on day one
   is how false positives become support tickets.
3. Find how the app already receives webhooks from other providers (payment processor, email service). Mirror that
   structure: raw-body access, signature check, quick acknowledgment, work handed to a queue or job.

## Build the receiver

1. An endpoint that reads the **raw** request body as a string before any JSON parsing. Frameworks that parse JSON
   by default need an exception for this route, or the signature will never match.
2. Verify the `X-Dregs-Signature` header: HMAC-SHA256 of the raw body with the channel's signing secret, hex
   encoded, compared in constant time. Reject with 401 on mismatch. Read the secret from an environment variable;
   the user gets it once when they create the channel, and it is not the `sk_` API key.
3. Check `X-Dregs-Timestamp` is recent (a few minutes) to blunt replays, and use the event's identifiers to make
   handling idempotent, since Dregs retries failed deliveries.
4. Dispatch on the `event` field in the body (also in `X-Dregs-Event`): identity scored, badges changed, escalation
   opened, escalation closed. Return 2xx quickly and do the real work asynchronously; a slow endpoint gets retried.
5. Implement the agreed response for each event type. The payload's `identityId` is the same stable ID the app
   passed to `dregs.identify()` and in backend events, so look the user up by it. Scores are integers from 0 to 100,
   higher is better.

## Create the channel and connect it

Only the user can create a webhook channel, under **Settings → Notifications** in the dashboard, choosing the
endpoint URL and the events to subscribe to; the signing secret is shown once at creation. Then:

- `list_channels` to find the new channel's slug and confirm it is enabled.
- `test_channel` (admin) to send a test delivery, and `list_channel_deliveries` to see whether it was delivered or
  failed and with what status code.
- For escalation events, the channel has to be attached to an escalation rule. `list_escalation_rules` shows the
  rules; `update_escalation_rule` (admin, with confirmation) adds the channel slug to the ones that should notify.
  The `tune-rules` skill covers writing new rules.

## Verify end to end

With the endpoint deployed to an environment Dregs can reach (a tunnel is fine for development), trigger a
delivery with `test_channel`, then `analyze_identity` on a test identity to produce a real `identity.scored`
delivery, and confirm the app recorded or flagged as designed. Check `list_channel_deliveries` for anything
FAILED and fix the endpoint before moving on.

## Hand off

Summarize the endpoint, the environment variables (signing secret per channel, per environment), the events
subscribed, and the response behavior. Point the user at the `tune-rules` skill to shape which escalations reach the
endpoint, and remind them that a webhook response is only as good as the rules feeding it.
