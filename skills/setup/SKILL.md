---
name: setup
description: Integrate Dregs into the user's application, end to end and under their supervision. Drives the setup-tracking, setup-events, and setup-webhooks skills in order, checking the account and confirming each step with the user. Use when the user wants to add Dregs to their app, integrate Dregs, install the tracking script, start sending events, or wire up webhooks, and it is not clear which part they need.
---

# Integrating Dregs into an application

A complete Dregs integration has four parts, and most of the value arrives with the first two:

1. **Tracking** (`setup-tracking`): the `dregs.js` snippet on every page, `dregs.identify()` at signup and login,
   and `dregs.track()` for the actions that matter. This is where device fingerprints and behavior come from.
2. **Backend events** (`setup-events`): server-side `POST /api/events` for actions the browser never sees, and the
   mappings that tell Dregs which of the user's event names and identity fields mean registration, login, email,
   and so on. Without the mappings, most analyzers stay quiet.
3. **Webhooks** (`setup-webhooks`): an endpoint that verifies Dregs's signature and does something with scores and
   escalations, so the integration acts instead of only reporting.
4. **Rules** (`tune-rules`): the badges and escalations that decide what "something" is. Usually a later step, once
   there is data to look at.

Every integration is specific to the user's stack, routes, auth flow, and event names. Your job is to walk them
through it, propose concrete changes to their code, and let them decide. Do not guess at their architecture when you
can look, and do not look at what you can ask.

## Before you start

1. **Confirm the connection.** Call `get_account_summary`. If it fails or the tool is missing, follow the `connect`
   skill first. Note the team name; everything you set up lands in that team.
2. **See what is already there.** `dashboard_stats` and `list_identities` tell you whether any data is flowing.
   `search_events` shows the event names in use. `list_mappings` (kind `EVENT_TYPE` and `IDENTITY_FIELD`, scope
   `CUSTOMER`) shows what is already mapped. An account with events but no identified users, or identified users but
   no mappings, needs a different starting point than an empty one.
3. **Look at the codebase.** Identify the framework, the templating or layout entry point where a script tag belongs,
   where signup and login complete, where server-side actions such as payments and password resets happen, and how
   the app handles environment variables and incoming webhooks today.
4. **Read the manual.** Call `get_documentation` with `GETTING_STARTED` now, and the chapter each sub-skill names
   before its step. The snippets in the manual are the ones to use; do not reproduce them from memory.
5. **Ask what they want.** Present the four parts with a one-line status for each (present, partial, missing) and ask
   which to do now. A first session usually covers tracking and backend events and stops; webhooks come once scores
   exist to act on.

## Credentials

The user creates API credentials in the Dregs dashboard under **Settings → Credentials**. Each credential has a
public key (`pk_…`, safe in frontend code) and a secret key (`sk_…`, server only, shown once). You cannot create
credentials, and you should never ask the user to paste a secret key into the conversation or write one into a file
that will be committed. Have them put the public key where the snippet needs it and the secret key in an environment
variable, and reference the variable in the code you write.

## Running the steps

Work through the chosen parts in order, one sub-skill at a time. For each:

- Explain what will change and why, then make the change as an edit the user can review, or as a proposed diff if
  they prefer to apply it themselves.
- Verify with Dregs before moving on. Each sub-skill ends with a check through the MCP tools (events arriving,
  identities identified, mappings in place, a test delivery received). A step is not done until Dregs confirms it.
- Keep a short running summary of what was added and where, so the user can find it later.

## Finishing

Summarize the integration: files touched, environment variables the user must set in each environment, the events and
identity fields now flowing, the mappings applied, and what remains. Suggest the `health-check` skill for a full
diagnosis after a day of traffic, and the `tune-rules` skill once there are scores worth acting on.
