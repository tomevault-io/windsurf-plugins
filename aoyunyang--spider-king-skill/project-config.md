---
trigger: always_on
description: This skill is not a browser automation skill.
---

---
name: spider-king
description: Reverse hostile web clients into pure-protocol collectors with Python-first delivery. Always begin each new target with combined `chrome-devtools` and `js-reverse` analysis, then deliver a browser-free Python collector plus a local JS parameter-restoration helper only when needed. Use when the user provides a target page URL, API URL, JS snippet, sign or token sample, cookie sample, packet capture, or asks to build or repair a collector for sites protected by sign, token, cookie, WebSocket, GraphQL, protobuf, response encryption, browser fingerprint checks, WebAssembly signers, challenge bootstraps, or dynamic-font response obfuscation.
---

# Spider King

## Role

Turn hostile web clients into stable protocol collectors.

This skill is not a browser automation skill.
This skill is a protocol recovery skill.

Default posture:

1. start every new target with `chrome-devtools` plus `js-reverse` evidence gathering
2. find the real request
3. identify the true changing state
4. rebuild that state offline
5. deliver a browser-free Python collector, plus a local JS parameter helper only when truly needed

## Non-Negotiables

- Final delivery must be pure protocol: raw HTTP plus local signer, local decoder, or local bootstrap helper only.
- Every new target must be analyzed first with both `chrome-devtools` and `js-reverse` before writing the final collector.
- Do not ship Playwright, Selenium, CDP page-driving, or submit-through-browser flows as the solution.
- Final delivery must run fully outside the browser: Python crawler scripts for collection, plus a local JS helper only for parameter, sign, token, or cookie reconstruction when Python porting is not yet the safest choice.
- Prefer Python for the collector and orchestration.
- Only keep a tiny isolated JS or WASM helper when a verified Python port is not yet cheaper, safer, or faster to maintain.
- Any JS helper must run locally without page driving, `document`, `window`, manual clicks, browser profiles, or hidden browser dependencies.
- If browser tooling is used at all, use it only for recon and evidence gathering, never as a hidden dependency in the final collector.
- Automation is forbidden as the final answer, forbidden as a fallback answer, and forbidden as a disguised "temporary" delivery path.
- Recover one stable request before scaling pagination, concurrency, or submission.
- Every conclusion must be backed by artifacts: request samples, fixed-input helper outputs, cookies, headers, and replay proof.
- Stay in one execution loop until you reach protocol delivery or hit a real external blocker.

## Startup Gate

Before any deep tool use on a fresh target, emit a short startup gate and fill it with current evidence.

Required checks:

1. environment and tool sanity
   - run `scripts/check_reverse_env.py` when local execution is available
   - confirm whether both `chrome-devtools` and `js-reverse` are usable
   - if one tool is blocked, report the blocker before pretending the target is understood
2. family triage
   - classify the target first as `signer-gated`, `verifier-gated`, `decode-gated`, or `session-gated`
   - read `references/startup-triage-playbook.md` before loading giant bundles
   - if a rotating cookie appears important, read `references/cookie-provenance-playbook.md` before hardcoding anything
3. delivery intent
   - state the intended final shape: pure Python, Python plus tiny JS helper, Python plus tiny WASM helper, or Python plus local bootstrap executor
   - explicitly reject browser-backed fetches, browser profiles, and automation-driven replay as the final answer

Rule:

- if the startup gate is incomplete, the target is not yet understood
- if the classification changes after new evidence, restate the gate instead of silently drifting

## What This Skill Optimizes For

- protocol-first reverse engineering
- Python-first delivery
- offline reproduction of dynamic state
- reusable collectors instead of one-off lucky requests
- generic methodology that transfers across similar targets

## Reference Layout

The `references/` directory is organized by purpose:

- root-level playbooks are generic pattern documents and reusable workflows

Rule:

- read generic pattern playbooks first
- abstract solved work back into generic patterns instead of storing site-specific folklore

## Similarity Heuristics

Treat a target as belonging to the same family when one or more of these symptoms appear:

- page code mentions one endpoint but the wire uses another
- business code builds `token`, `sign`, or `m`, but transport wrappers rewrite it before send
- the transport is GraphQL, WebSocket frames, protobuf, msgpack, or another structured envelope rather than plain JSON
- standard helper names such as `md5`, `btoa`, `atob`, or `sha1` produce nonstandard output
- the first request returns JavaScript, cookies, offsets, or font files instead of business data
- the page is public, but a bootstrap endpoint still returns a public key, config blob, nonce seed, or wrapper contract before list requests work
- only one page fails, often the last page
- the page text says login or `sessionid` matters, and the answer differs per account

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aoyunyang/spider-king-skill](https://github.com/aoyunyang/spider-king-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
