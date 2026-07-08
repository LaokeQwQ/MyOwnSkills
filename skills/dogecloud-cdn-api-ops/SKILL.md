---
name: dogecloud-cdn-api-ops
description: Work with DogeCloud CDN APIs for domain listing, cache refresh, and related operational debugging. Use when integrating or troubleshooting DogeCloud CDN refresh APIs, request signing, domain ownership errors, or admin-panel CDN automation.
---

# DogeCloud CDN API Ops

Use this skill when a project integrates with DogeCloud CDN APIs, especially:

- listing CDN domains
- submitting refresh tasks
- querying refresh status
- debugging signature or domain errors
- wiring DogeCloud refresh into admin panels or deployment flows

## Core rules

- Prefer DogeCloud’s documented request format.
- For CDN refresh APIs, prefer `application/x-www-form-urlencoded` unless there is a strong reason not to.
- Sign the request body exactly as sent.
- Verify domain ownership and status before debugging more exotic causes.

## Common APIs

- `GET /cdn/domain/list.json`
- `POST /cdn/refresh/add.json`
- `POST /cdn/refresh/query.json`

## Signing pattern

For DogeCloud API requests:

- signature input is:
  - `apiPath + "\\n" + requestBody`
- use:
  - `HMAC-SHA1`
- authorization header format:
  - `TOKEN <ACCESS_KEY>:<signature>`

Important:

- The request body used for signing must match the actual transmitted body exactly.
- If using form encoding, sign the encoded string.
- If using JSON, sign the exact JSON string.

## Refresh API guidance

For `/cdn/refresh/add.json`:

- use `rtype=url` for URL refreshes
- pass `urls` as a form field
- multiple URLs can be sent as a comma-separated string
- keep batch sizes conservative

## Domain ownership debugging

If DogeCloud returns errors like:

- `提交的 URL 中没有找到属于你的域名`

check in this order:

1. The target domain exists under the current account.
2. The domain status is `online`.
3. The exact hostname matches what was added in DogeCloud.
   - `example.com` and `www.example.com` are different domains.
4. The submitted URL starts with `http://` or `https://`.
5. The request format matches the documented format.
6. Non-ASCII URL parts are URL-encoded.

## URL encoding rule

- Encode URLs containing Chinese or other non-ASCII path segments before refresh submission.
- `encodeURI(...)` is usually appropriate for whole-page URLs being refreshed.

## Recommended debugging flow

1. Query `domain/list.json` with the same credentials used by the app.
2. Confirm the target hostname is present and `online`.
3. Test a single refresh on the site root URL manually.
4. If manual refresh works but app refresh fails, compare:
   - body format
   - content type
   - URL encoding
   - credentials actually used at runtime

## Admin-panel integration guidance

When integrating DogeCloud into an admin panel:

- keep credentials in env vars
- log the effective refresh target count
- show separate build logs and CDN logs
- if refresh fails, surface the exact DogeCloud `msg`
- batch large URL lists
- URL-encode post URLs before submission

## Safety note

- If API keys or secret keys are exposed in chat, logs, screenshots, or commits, instruct the user to rotate them immediately.
- Do not keep using leaked credentials as if nothing happened.

## Verification checklist

After changes:

- the app can list domains under the intended DogeCloud account
- a single root URL refresh succeeds manually
- automated refresh uses the same account and target hostname
- multi-URL refresh uses encoded URLs and documented body format
