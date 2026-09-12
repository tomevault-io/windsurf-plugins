---
trigger: always_on
description: <!-- Generated from AGENTS.md by scripts/sync-rules.mjs. Do not edit directly. -->
---

<!-- Generated from AGENTS.md by scripts/sync-rules.mjs. Do not edit directly. -->

# mSpace Integration — Agent Instructions

> Portable entry point for Cursor, Windsurf, GitHub Copilot, Codex, Cline, Aider, Zed and any
> other agent that reads `AGENTS.md`. Claude Code and the Agent SDK use [SKILL.md](SKILL.md) —
> same content, skill frontmatter.

mSpace is Mobitel's application platform for **Sri Lanka**. It exposes SMS, USSD, subscription
management, mobile-account charging, OTP verification and location as JSON-over-HTTPS APIs.

It has two tracks: **Inzpire**, the API track this skill is about, and **Xpand**, a no-code track
for Contact, Vote, Alert and Scheduled Messages applications. If the requirement is fully covered
by an Xpand template, say so rather than building an integration.

**Apply these instructions whenever the work involves mSpace, Inzpire, Xpand, `api.mspace.lk`,
`tel:` MSISDN addressing, USSD menus, short code and keyword routing, subscriber base size,
mobile-account charging, or telco SMS in Sri Lanka.**

The platform is JSON over HTTPS, so **any language builds a complete integration** — write it in
whatever the host project already uses. Every endpoint is written out as a runnable curl, with its
parameters and its response defined, in
[references/14-curl-reference.md](references/14-curl-reference.md): translate the request into the
project's HTTP client and you have the call, whatever the language. There is no code generator
here on purpose — a curl is the same call in every language, where an emitter would serve six and
age with their idioms. [references/12-any-stack.md](references/12-any-stack.md) specifies the
surrounding integration language-neutrally, and reference implementations exist for
TypeScript/Node, Python, Java, Go, PHP and C# as worked examples.

---

## Non-negotiable rules

1. **Never hardcode `applicationId` or `password`.** Environment variables only, read through one
   config module, validated at startup. Not in source, not in a client bundle, not in a committed
   file, not in a log, not in git history. For CaaS the password is the API key mailed to you on
   application approval — move it out of that inbox.
2. **Never call mSpace from client-side code.** Browsers and mobile apps call *your* backend; your
   backend calls mSpace. The credentials are a shared secret and the platform enforces the
   *Allowed Host Address* list.
3. **Never subscribe or charge without explicit, recorded consent**, and never without disclosing
   the amount and frequency first. Both are provisioning-level settings on the application.
4. **Never assume `subscriberId` is a real phone number.** With Mobile Number Masking enabled it
   is a masked value. Store and send back exactly what you received.
5. **Charging is idempotent on `externalTrxId`**, persisted *before* the call and reused unchanged
   on any resolution attempt. Never retry with a fresh one.
6. **`S1000` is not the only success code.** CaaS OTP generation succeeds with **`P1003`**, and
   Subscriber List treats **`S1001`** ("No Subscribers Found") as a success.
7. **Every field is a string on the wire except `requestPage`.** `amount`, `otp`, `referenceNo`,
   `externalTrxId`, `action`, `encoding` and `deliveryStatusRequest` all look numeric and are all
   strings. Serialised as JSON numbers they corrupt the body silently — `"5.00"` becomes `5`, an
   OTP loses its leading zero, a 31-digit `referenceNo` becomes `8.8e+30`.

---

## Query the API catalog instead of guessing

This repo ships the complete mSpace contract as structured data
([`catalog/mspace-api.json`](catalog/mspace-api.json)) plus a zero-dependency CLI over it.
**Run these instead of recalling parameter names** — they are offline, read-only, need no install,
and never see credentials.

```bash
node tools/mspace.mjs list [category]              # every service and callback
node tools/mspace.mjs show <id>                    # full contract: params, response, rules
node tools/mspace.mjs search "<query>"             # find by intent, e.g. "base size"
node tools/mspace.mjs curl <id> [key=value ...]    # runnable request + param/response defs
node tools/mspace.mjs validate <id> '<json>'       # check a payload against the spec
node tools/mspace.mjs response <id> '<json>'       # read a real response against the contract
node tools/mspace.mjs code <statusCode>            # decode a status code + the fix
node tools/mspace.mjs diagnose "<symptom>"         # cause and fix from a symptom
node tools/mspace.mjs practices [severity]         # security and reliability rules
node tools/mspace.mjs checklist                    # go-live checklist
node tools/mspace.mjs reference <doc>              # print a reference document
node tools/mspace.mjs platform                     # base URL, tracks, operator, conventions
```

Add `--json` to any command for machine-readable output. If you cannot run commands — or Node is
not installed — read `catalog/mspace-api.json` directly; it is plain JSON with the same data. The
CLI is a documentation reader, not part of the integration, and constrains nothing about the stack
you build in.

**Use them in this order:** `search` or `list` to find the service → `show` for the exact

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hSenidMobileCPaaS/mSpace-as-a-skill](https://github.com/hSenidMobileCPaaS/mSpace-as-a-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
