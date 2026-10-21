# Disposable email spot check

Use this only to spot-check an address or a domain while you are writing the plan. It is the anonymous endpoint behind the [Disposable Email Checker](https://dregs.com/tools/disposable-email-checker/). No account and no API key.

Do not put signup traffic through it. The published cap is 10 requests per minute and 100 requests per day per IP, and that IP is shared with everyone else calling from the same network. A production signup form will blow the cap, and a timeout must not lock out a real signup. For a block you run on every signup, host a list yourself. The tool page names the community [disposable-email-domains](https://github.com/disposable-email-domains/disposable-email-domains) project as one upstream feed. That file is not a mirror of Dregs's list. Dregs curates its own lookup (those feeds plus its own signals, described on the tool page as over 8,000 domains, refreshed weekly). A spot check and a local file can disagree. When a block decision is close, trust a fresh spot check of that domain over a stale file, and still treat "not listed" as "not a known burner."

## Budget for one session

- At most 5 calls.
- One address or one domain per call. Send the field you mean.
- Only check a value the user gave you, or a domain they named as abuse (`mailinator.com` and the like).
- No CSV, no database export, no loop over a signup table.
- If the user hands you a file of addresses, refuse the bulk run. Check up to five domains that would change the advice, and tell them why you stopped.
- On HTTP 429, or any response that says the rate limit was hit, stop for the session. Do not retry, back off in a loop, or move the call somewhere else to dodge the cap.
- There is no key to rotate. Do not ask the user for one.

## Request

`POST https://dregs.com/api/tools/authenticity-check`

Header: `Content-Type: application/json`

No `Authorization` header.

Body, one of:

```json
{"email": "ada@gmail.com"}
```

```json
{"domain": "mailinator.com"}
```

If both `email` and `domain` are non-empty, the server checks `domain` and echoes `domain`. Confirmed with a live call on 2026-10-21: sending `a@gmail.com` plus `mailinator.com` returned `disposableEmailDomain: true` and a null `email`. Send one field.

Other identity fields (`firstName`, `lastName`, `fullName`, `username`, `nickname`, `organizationName`) are ignored on an anonymous call. The server lists them on the `X-Dregs-Ignored-Fields` response header and still returns the disposable verdict for the email or domain. Do not send them. They do not improve the anonymous result.

## Response

Anonymous success is HTTP 200 and `application/json`.

Live response for `{"domain":"mailinator.com"}` on 2026-10-21:

```json
{
  "input": {
    "email": null,
    "domain": "mailinator.com",
    "firstName": null,
    "lastName": null,
    "fullName": null,
    "username": null,
    "nickname": null,
    "organizationName": null
  },
  "disposableEmailDomain": true
}
```

Live response for `{"email":"ada@gmail.com"}`:

```json
{
  "input": {
    "email": "ada@gmail.com",
    "domain": null,
    "firstName": null,
    "lastName": null,
    "fullName": null,
    "username": null,
    "nickname": null,
    "organizationName": null
  },
  "disposableEmailDomain": false
}
```

The [tool page](https://dregs.com/tools/disposable-email-checker/) shows the same two fields with the nulls omitted. The live API includes the nulls. Read `disposableEmailDomain` and the field you sent. Ignore nulls.

How to read it:

| What you see | What it means |
| --- | --- |
| `disposableEmailDomain: true` and the domain or email you sent is echoed | That domain is on Dregs's disposable list. Blocking that domain is a list rule, not a reason to add a judgment product. |
| `disposableEmailDomain: false` and the value you sent is echoed | The domain is not on the known burner list. New burner domains show up between refreshes. This is not proof the signup is a real customer. |
| HTTP 200, and both `input.email` and `input.domain` are null | The check did not run. An empty `{}` body does this, and it still returns `disposableEmailDomain: false`. Do not call that a clean email. |
| HTTP 400, body like `{"status":400,"error":"Bad Request","message":"Request body is missing or malformed.","path":"/api/tools/authenticity-check",...}` | The body was not JSON. Fix the request. One retry of a corrected body is enough. |

Every anonymous 200 also sends `X-Dregs-Upgrade`. On 2026-10-21 that header advertised a free-plan allowance for authenticated calls. Do not repeat that number, and do not treat the header as the quota. The limit this skill follows is the one published on the tool page for anonymous calls: 10 per minute and 100 per day per IP.

## Authenticated calls are out of scope

The same URL accepts `Authorization: Bearer` with a Dregs secret key. The tool page says those calls can take name and organization fields and return an Authenticity score plus observations, at higher limits. This skill does not use that tier. Do not ask for a secret key, do not put one in a command line, and do not design the plan around it.

If the user already has a key and wants the fuller check, point them at the tool page and stop. That work is optional and is not part of this plan.
