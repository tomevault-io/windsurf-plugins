---
trigger: always_on
description: You can place REAL outbound phone calls by sending HTTP requests to the
---

# PlaceCall — place real phone calls (agent instructions)

You can place REAL outbound phone calls by sending HTTP requests to the
PlaceCall API with `curl`. There is no native voice tool — use the API. When
asked to call / ask a business / book / cancel by phone, DO IT via these calls.
No phone number at hand ("find me a florist and call them")? Use
`POST /v1/places/suggest` (below) to find the place — do NOT web-search.
(This is the same reference as `SKILL.md`, for Codex/AGENTS.md-based agents.)

## Connection
- Base: `https://api.voygr.tech`
- Auth: header `X-API-Key: $PLACECALL_API_KEY` on EVERY request. Never echo the key.
- Key unset? Source it in the SAME command as the call:
  `. ~/.codex/placecall.env && curl ...` (an `export` does not carry between
  commands). No such file: send the user to /checkout. Never glob for `.env`
  files, which is credential hunting and sandboxes refuse it.
- No key? Self-serve at <https://api.voygr.tech/checkout?src=gh-repo> ("Get free
  API key", emailed; that page also shows what a new key includes and the
  current rates). Recovery: `/recover`.
- Check: `curl -s -H "X-API-Key: $PLACECALL_API_KEY" https://api.voygr.tech/users/me`
- Surface marker: every `POST /calls` below carries `-H "X-Client-Surface: gh-repo"`.
  Keep it as written — it tells PlaceCall which listing these instructions came
  from (telemetry only; never affects auth, billing or the call).
- Client-agent marker: every `POST /calls` below also carries
  `-H "X-Client-Agent: ${CLAUDECODE:+claude-code}${CURSOR_AGENT:+cursor}${CODEX_SANDBOX:+codex}${GEMINI_CLI:+gemini-cli}"`.
  Each tool exports a distinct env var, so this expands to which tool placed the
  call (`claude-code`/`cursor`/`codex`/`gemini-cli`) — telemetry only, sibling to
  `X-Client-Surface`. If none is set the header is empty and PlaceCall falls back
  to the User-Agent.

## Place a call — one endpoint, everything in the brief
```sh
curl -s -X POST https://api.voygr.tech/calls \
  -H "X-API-Key: $PLACECALL_API_KEY" -H "Content-Type: application/json" \
  -H "X-Client-Surface: gh-repo" \
  -H "X-Client-Agent: ${CLAUDECODE:+claude-code}${CURSOR_AGENT:+cursor}${CODEX_SANDBOX:+codex}${GEMINI_CLI:+gemini-cli}" \
  -d '{"target_phone":"+1XXXXXXXXXX","brief":"<the full task in plain English>","language":"en","ask_user_mode":"stream"}'
# -> 201 {"call":{"call_id":"...","status":"dialing",...},"credits_reserved":30,...}
```
- `target_phone` E.164 (required); `brief` (required, max 4000 chars) = the ONLY
  thing the bot reads — put every detail (what to ask, who for — there is no
  separate caller-name field — names/dates/party size/callback number, how to
  wrap up). `language`: 13 codes accepted (en/es/fr/de/hi/ru/pt/ja/it/nl/sr/tr/pl)
  or `auto` (default → en); anything else is a fast 422. `en` is the most
  reliable; non-English is best-effort. US numbers only. Booking/cancel =
  describe it in the brief.
- **Always send `"ask_user_mode":"stream"`** — otherwise mid-call `ask_user`
  questions may be routed elsewhere and never reach your events poll loop.
- Capture the call id: **nested at `call.call_id`** on this freeform path.
- Typed alternative (same endpoint): send `intent` + `slots` instead of `brief`.
  Five intents: `inquiry`/`info_gathering`/`issue_resolution`/`booking`
  (name, date, time, party_size)/`cancellation` (`booking_id`, no phone). A 422
  `missing_slots` lists each gap with a `suggested_question` — refill and
  resubmit. This path returns a FLAT envelope (top-level `call_id`). Schema:
  `GET /skills/{id}/manifest`. **Prefer it for bookings** — slots are checked
  before anything is dialed, and the optional `phone_to_dictate` slot gives the
  bot a callback number to read back (staff ask for one more often than not).

## No number? Find the place first — `POST /v1/places/suggest`
Free-text need → up to 4-6 ranked place cards, each ready to dial. Billed:
5 credits per answered request (a degraded answer and a short list bill in
full; a refusal bills nothing), from the same balance as calls, with a
refundable hold larger than the charge — so `402` can fire while the balance
still covers the 5. Rate is operator-set; see
<https://api.voygr.tech/checkout>. Own limits: 10/min, 1000/UTC-day,
separate from call limits.
US only, English in/out.
```sh
curl -s -X POST https://api.voygr.tech/v1/places/suggest \
  -H "X-API-Key: $PLACECALL_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"florist in Chicago with fresh peonies in stock today",
       "location_hint":"Wicker Park","booking_name":"Alex","callback_phone":"+13125550188"}'
```
Only `query` required; `location_hint` needed for "near me" wording;
`booking_name`/`callback_phone` get baked into every card's `call_brief`.
Response: `{request_id, intent, suggestions: [{suggestion_id, rank, name,
address, phone_e164, website, confidence, price_band,
open_at_target, why, product_match, verify_on_call, call_brief, call_ready}],
degraded, degradation_reason, short_list_reason}` — cards ordered by `rank`,
each with its OWN `suggestion_id`. `short_list_reason` explains a short list:
`"thin_pool"` = market is thin (comes with `degraded: true`/`few_results`);
`"curated"` = ranker deliberately picked fewer from a full pool (good sign,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [voygr-tech/placecall](https://github.com/voygr-tech/placecall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
