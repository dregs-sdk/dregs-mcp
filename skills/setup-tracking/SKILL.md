---
name: setup-tracking
description: Add the Dregs tracking script to the user's frontend, identify users at signup and login, and track key actions, then verify events arrive. Use when the user wants to install dregs.js, add the tracking snippet, call dregs.identify, or asks why Dregs shows no identities or only anonymous ones.
---

# Setting up frontend tracking

Frontend tracking is the source of device fingerprints, sessions, and browser behavior, which drive the humanity and
uniqueness categories. It is three pieces: the snippet on every page, `dregs.identify()` when you know who the user
is, and `dregs.track()` for the actions worth scoring.

## Before you change anything

1. Call `get_documentation` with `TRACKING` and read the frontend section. The snippet, `dregs('init', …)`
   options, and the `identify` and `track` signatures there are authoritative.
2. Find where a script tag reaches every page: a base layout, an `index.html`, a `_document` or root layout, or the
   head partial. If the app has several entry points (marketing site, app shell, embedded widget), say so and ask
   which should carry the snippet.
3. Find where signup and login complete on the client, and where the app already knows the user's stable ID. The
   ID passed to `identify` must be the same one the backend will use in events later, so pick it deliberately: the
   database user ID, not an email or a session token.
4. Ask the user for the public key (`pk_…`) from **Settings → Credentials**, or have them fill it in themselves. The
   public key is safe in frontend code. Never ask for the secret key here.

## Install the snippet

Add the loader snippet from the manual to the shared layout, with the public key in `dregs('init', …)`. If the app
has a content security policy, note that `https://dregs.com` must be allowed for scripts and connections, and show
the exact directive change rather than loosening the policy.

## Identify users

Call `dregs.identify(id, data)` after signup and after login, as early as the client knows the user. Pass the stable
ID and the profile fields the user is willing to share: name, email, username, plan, and anything else that helps
authenticity scoring. Use the app's existing field names; the `setup-events` skill maps them to Dregs's canonical
fields afterwards. If the app renders the logged-in user into the page on load, identify there too, so returning
sessions are attributed without waiting for another login.

## Track key actions

Add `dregs.track(type, data)` for the actions that carry signal and that the browser sees: signup completion,
login and failed login, password reset requests, email confirmation, profile changes, referral or promo use,
checkout and payment attempts, and the app's core actions (posting, inviting, exporting). Use the app's own event
names and keep them consistent with the backend names. Do not track page views by hand; the script already collects
what the manual's "What's Collected Automatically" section lists.

## Verify

Have the user load the app, sign in, and perform one tracked action. Then:

- `list_identities` should show the identified user with a display name or email, not an anonymous identity.
- `search_events` filtered to that identity should show the tracked events with their names and data.
- `list_devices` should show at least one device with a fingerprint and geo context.

If events arrive but identities stay anonymous, `identify` is not running, or runs before the script initialized;
check the call order. If nothing arrives, check the browser console for a blocked script (CSP or an ad blocker) and
that the key is a `pk_` key from the right team.

## Hand off

Tell the user which files changed, that the public key must be set per environment, and that the `setup-events`
skill is next: backend events and, above all, the mappings that let analyzers recognize what the events mean.
