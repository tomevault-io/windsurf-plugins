---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm test                              # all tests (node:test, zero dependencies)
node --test tests/logic.test.js       # one file
node --test --test-name-pattern "streak"   # one test by name
npm run icons                         # regenerate icons/ from tools/make-icons.mjs
npm run package                       # dist/aileethub-v<version>.zip for the web store
```

There is no bundler, transpiler, linter, or install step — `node_modules/` is empty by
design and `npm test` runs against the same files Chrome loads. Load the extension with
**Load unpacked** at `chrome://extensions` pointed at the repo root.

`package.json` exists only for the test/tooling scripts and to make Node treat `.js` as
ESM; Chrome ignores it. `npm run package` refuses to build when `package.json` and
`manifest.json` versions disagree — bump both.

Reloading the extension does **not** reload content scripts. After touching anything in
`src/content/`, reload at `chrome://extensions` *and* refresh the LeetCode tab.

## Architecture

A single data flow, split across three execution contexts by what each one is allowed
to touch:

```
LeetCode page (MAIN world)      interceptor.js   patches fetch + XHR
        │ window.postMessage
LeetCode page (ISOLATED world)  bridge.js        GraphQL enrich, toast UI,
        │                                        LeetCode fetch proxy
        │ chrome.runtime.sendMessage
Service worker (module)         service-worker.js  live sync
                                backfill.js        history import
```

**Why the split matters:** content scripts never see the GitHub token or Groq key.
Every call carrying a secret happens in the service worker. Keep it that way — do not
move `github.js` or `groq.js` calls into `src/content/`.

### Detection (`src/content/interceptor.js` + the poll in `bridge.js`)

The interceptor monkey-patches `window.fetch` and `XMLHttpRequest.prototype.open`/`send`
in the page's own world (`"world": "MAIN"`, `run_at: document_start` — it must patch
before the page's first call). It has no `chrome.*` access and must stay a plain script
with no imports.

**Detection is deliberately redundant, because LeetCode has moved the result panel
between transports before and single-signal detection died silently when it did.** Three
signals go over one `window.postMessage` channel:

| Signal | Source | Role |
| --- | --- | --- |
| `submitted` | `POST …/submit/` → `submission_id` | **Load-bearing.** Fires on the request that *starts* every submission |
| `accepted` | `GET /submissions/detail/<id>/check/` → `SUCCESS` + `status_code 10` | Fast path |
| `accepted` | `POST /graphql/` `submissionDetails` → `statusCode 10` | Fast path |

`submitted` carries no verdict, so `bridge.js` polls the judge itself (`waitForVerdict`,
same-origin so the session cookie applies) and pushes only on an accept. That is what
makes detection survive a transport change: any future result mechanism still starts
with a `/submit/` call. The fast paths only skip the polling when they happen to fire.
A rejected submission and a poll timeout are both **silent** — no toast.

`bridge.js` also carries a DOM fallback for the case where interception sees nothing at
all: `[data-e2e-locator="submission-result"]` reading "Accepted" recovers the id from
`questionSubmissionList`. It waits 4s first so the interceptor, which knows the exact id,
wins. Dedupe across all four routes is the `handled`/`inFlight` pair keyed by submission id.

Set `localStorage['ailh:debug'] = '1'` on leetcode.com to trace both scripts.

### Enrichment (`src/content/bridge.js`)

Runs in the isolated world, so it has `chrome.*` *and* shares the page's cookies for
same-origin requests. It re-fetches authoritative data over LeetCode's GraphQL API
(`submissionDetails` for code/lang/perf, `questionData` for difficulty/tags/statement),
because the check endpoint's payload is inconsistent. Interceptor values are fallbacks
only. Requires the `csrftoken` cookie as an `x-csrftoken` header.

### Orchestration (`src/background/service-worker.js`)

`pushSubmission()` is the whole pipeline. Two invariants to preserve:

1. **A Groq failure must never cost the user their commit.** It is caught, the README
   falls back to a template, and the toast says "without AI notes".
2. **A root-README index failure must never cost the user their commit either** — same
   pattern, the index file is just dropped from the commit.

### History import (`src/lib/backfill.js`)

Imports already-solved problems, one commit each, **backdated to the original solve
time**. Four things here are load-bearing:

1. **All LeetCode calls are proxied through a tab.** LeetCode's session cookie is
   `SameSite`, so a `fetch` from the service worker's origin arrives signed out.
   `ensureTab()` finds or opens a leetcode.com tab and sends `LEETCODE_FETCH` to
   `bridge.js`, which performs the request in the page's origin. Do not "simplify" this
   into a direct fetch — it fails silently with an empty history rather than an error.
2. **Progress lives in storage, not memory.** An MV3 worker is killed when idle, so

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byRudra/AILeetHub](https://github.com/byRudra/AILeetHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
