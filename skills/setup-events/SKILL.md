---
name: setup-events
description: Send server-side events to Dregs and map the user's event names and identity fields to Dregs's canonical types. Use when the user wants to track backend actions like payments or password resets, asks how to send events from their server, or when Dregs is receiving events but scores sit in the middle because analyzers cannot tell what the events mean.
---

# Setting up backend events and mappings

Two jobs, and the second matters more than most people expect. Backend events give Dregs the actions only the server
sees. Mappings tell Dregs which of the user's event names mean registration, login, password reset, and so on, and
which identity fields hold the email, name, and username. Analyzers key off those canonical types; unmapped events
are still stored but contribute little.

## Backend events

1. Call `get_documentation` with `TRACKING` and read the backend section, and with `EVENTS` for what to track.
   The request shape (`type`, optional `data`, `identity` with `id` and optional `data`, optional idempotent `id`)
   is defined there.
2. Find where the server performs actions worth scoring and that the browser does not see: payment charges and
   failures, refunds, subscription changes, password resets and changes, MFA changes, role and permission changes,
   invitations, API-key usage, and background jobs the user triggered. Avoid double-counting actions the frontend
   already tracks.
3. Use the official Dregs SDK for the app's language. Writing an HTTP client by hand is the fallback, for languages
   that do not have one yet.

   | Language | Package | Install |
   | --- | --- | --- |
   | Python 3.10+ | `dregs` on PyPI | `pip install dregs` or `uv add dregs` |
   | TypeScript and Node 20+ | `@dregs/sdk` on npm | `npm install @dregs/sdk` |
   | Java 17+ | `com.dregs:dregs-sdk` on Maven Central | add the Gradle or Maven coordinate |
   | Ruby 3.1+ | `dregs` on RubyGems | `bundle add dregs` |
   | PHP 8.2+ | `dregs/dregs-sdk` on Packagist | `composer require dregs/dregs-sdk` |

   On npm the bare `dregs` package is the browser tracking script, not the SDK; the server SDK is `@dregs/sdk`. On
   PyPI and RubyGems the bare `dregs` name is the SDK. Each SDK's source is at
   `github.com/dregs-sdk/dregs-sdk-<language>`.

4. Construct the client with the secret key (`sk_…`) read from an environment variable. The SDKs send it as a bearer
   token and are server-side only; the public key (`pk_…`) stays with the frontend script.

   All five expose the same surface, named to each language's idiom: `track(type, identity, data, identity_data,
   event_id, timestamp, source)`, `identities.get(id)`, `identities.scores(id)`, `identities.analysis(id)`,
   `identities.analyze(id)`, and webhook signature verification. `identities.scores()` returns the four category
   integers, which is the cheap read most integrations want; `identities.analysis()` returns the observations behind
   them. Typed errors per status (400, 401, 402, 404, 429, 5xx), retries with backoff, and an idempotency id on every
   event are built in, so do not rebuild them.

5. Call it from the places found in step 2, sending the same stable user ID the frontend passes to
   `dregs.identify()`, and the app's own event or transaction ID as the event id wherever one exists so a retry
   cannot double-count; the SDK generates one when you do not supply it. Pass profile data as identity data where the
   server knows something the browser does not (plan, account age, verified phone). Make the call fire-and-forget or
   queued; a Dregs outage must not fail the user's request.
6. Only if no SDK covers the language, write one small client for `POST /api/events` with the request shape from step
   1, and add the retries and the idempotent `id` yourself.
7. Never write the secret key into a file that will be committed, and never ask the user to paste it into the
   conversation. They set the variable; you reference it.

Verify with `search_events` filtered to a test user: the backend events appear alongside the frontend ones under the
same identity. If they appear under a different identity, the IDs do not match; fix the ID before anything else.

## Mappings

1. Call `get_documentation` with `EVENTS` if you have not, then `list_mappings` with kind `EVENT_TYPE` and scope
   `GLOBAL` to see the canonical event types and the names Dregs already recognizes by default, and with scope
   `CUSTOMER` to see what this team has mapped. Do the same for kind `IDENTITY_FIELD`.
2. Discover what the app actually sends. `search_events` over recent traffic gives the distinct event names in use;
   `get_identity` on a few identified users shows the attribute keys the app passes to `identify` and in
   `identity.data`. The codebase confirms both.
3. Propose the mapping table to the user: each event name to a canonical type (REGISTRATION, LOGIN, LOGIN_FAILED,
   PASSWORD_RESET, PASSWORD_CHANGE, EMAIL_CONFIRMATION, PURCHASE, PROMO_REDEEM, and the rest listed in the global
   mappings), and each identity field to a canonical field (EMAIL, FULL_NAME or FIRST_NAME and LAST_NAME, USERNAME,
   ORGANIZATION_NAME). Only map names that carry that meaning; an unmapped event is better than a wrong one.
   Distinguish PASSWORD_RESET (the recovery flow) from PASSWORD_CHANGE (a signed-in change), because they imply
   different attack paths.
4. Apply with `set_mapping` at scope `CUSTOMER` after the user confirms, one mapping per call. Never write to the
   GLOBAL scope; it belongs to Dregs. Mappings take effect on the next scoring pass, and `analyze_identity` (admin)
   on a test user shows the effect right away.

Verify with `list_mappings` at scope `CUSTOMER`, then `get_identity_analysis` on a test user after a re-score: the
observations should now reference registration, login, or email signals that were absent before.

## Hand off

Summarize the events now flowing from the server, the environment variable the user must set in each environment,
and the mappings applied. Suggest the `setup-webhooks` skill when the user is ready to act on scores, and the
`health-check` skill after a day of traffic to confirm analyzers are producing observations.
