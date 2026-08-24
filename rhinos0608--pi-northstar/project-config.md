---
trigger: always_on
description: Pi-Northstar is the Pi coding agent's browser/desktop automation and web search extension. It provides CDP-based browser control, agent-browser integration, desktop automation (via Cua Driver MCP), hybrid search (BM25 + vector embedding + RRF fusion), social/reach tools, and cookie/auth management.
---

# Agent Reference: Pi-Northstar

Pi-Northstar is the Pi coding agent's browser/desktop automation and web search extension. It provides CDP-based browser control, agent-browser integration, desktop automation (via Cua Driver MCP), hybrid search (BM25 + vector embedding + RRF fusion), social/reach tools, and cookie/auth management.

## Sister Repos

### No protocol-level dependencies on the other Pi repos.
Pi-Northstar is self-contained. It does not consume `@rhinos0608/pi-workspace-protocol`, Pi-SmartRead, or Pi-SmartEdit directly.

## Operational Contracts and Invariants

### Application SSRF guards (Scope A)
Public user-controlled fetch/browser URLs use `src/network-policy.ts` and reject private/reserved literals, metadata/local hostnames, credentials, and private DNS answers. Browser navigation also freezes allowed domains and performs system-DNS preflight. This is defense-in-depth, not complete SSRF containment; container egress remains authoritative.

Configured local SearXNG, Ollama, embedding, sidecar, CDP/setup paths remain operator-owned and bypass public validation (`unsafeFetchJson` is intentional). Loopback browser access is only through `browser-tools` → `LoopbackProxy`.

Residual risks: DNS rebinding, Chromium DNS TOCTOU, redirects, and debug-server outbound proxying. See ADR 0003.

### Python child processes MUST use the shared env allowlist
`src/python-child-env.ts` exports `buildPythonChildEnvironment()` — a sanitized environment with an allowlist of benign system/PI vars and a `BLOCKED_PATTERN` excluding TOKEN/KEY/SECRET/COOKIE/PASSWORD/API_KEY/API_SECRET/AUTH/BEARER and NODE_OPTIONS, NODE_PATH, PYTHONPATH, GIT_CONFIG_, SSL_CERT_, LD_PRELOAD, DYLD_ patterns.

**All three Python spawn sites use it:**
- `src/scrapling-bridge.ts:384` — `spawnProcess` (per-fetch Python bridge)
- `src/scrapling-bridge.ts:528` — `oneShotCommand` (health check)
- `src/sidecar-manager.ts:117` — `SidecarManager.start` (embedding sidecar)

**When adding a new Python child process:** always pass `env: buildPythonChildEnvironment()`. Never default to inheriting `process.env` — full environment leakage exposes API keys, cookies, and tokens to arbitrary code running in those child processes.

**Tests exist:** `test/python-child-env.test.ts` verifies the allowlist logic and confirms sentinel secrets do NOT leak through any of the three spawn sites (via mocked child processes). If the allowlist changes, these tests must pass.

### Desktop policy enforcement
`src/desktop-policy.ts:validatePolicy` now directly calls `isDeniedUpstreamTool` as defense-in-depth (not relying on upstream callers to check it). `test/desktop-policy.test.ts` covers all branches: allowed/denied actions, confirmation checks, pid/windowId validation, screenshot restrictions, stateId requirements.

### desktop-contract.ts constants (ground truth)
- `MAX_AX_NODES = 1000` (not 5000 — the README was wrong and has been fixed)
- `MAX_AX_DEPTH = 32` (not 50)
- `MAX_DIMENSION = 10000` (not 2048 — this is the desktop screenshot cap; browser screenshots have a separate `SCREENSHOT_MAX_DIMENSION = 8000` in `src/agent-browser.ts:74`)

### Browser tool schema limitations
The registered `browser` tool in `src/index.ts` does not expose `compact`, `semanticAction`, `job`, or `batch` parameters in its schema — but `src/browser-policy.ts:validateBrowserRequest` handles all of them. Agents using tool schemas for planning won't discover these features. This is a known gap.

### semanticAction uses `verb` and `query` fields (not `action` and `role`)
`src/browser-policy.ts:SemanticActionRequest` shape: `{ locator, query, verb, name?, index?, value?, exact? }`. The README code example was fixed to match; double-check any new docs or agent prompts that reference the old `{ action, role }` field names.

## Architecture Notes
- `src/native-tools.ts` is an ~900-line dispatcher mixing web search backends, semantic crawl, academic research, GitHub API, and embedding pipeline. This is the most coupled file in the codebase — refactoring it is deferred technical debt, not a quick fix.
- `src/reach-tools.ts` has a `case 'browser'` in `dispatchReachTool` that is **dead code** — the `browser` tool is registered independently in `src/index.ts` and never flows through this dispatcher. Do not add logic there expecting it to execute.
- `src/cdp.ts` is a self-contained CDP implementation (793 lines, zero external deps). It has no WebSocket connection-failure or protocol-error tests — if CDP behavior changes, those paths need coverage.
- The desktop control stack (`desktop-tools.ts` → `cua-client.ts`) is cleanly separated from search and browser modules with no cross-imports.

## Residual Risks
- **No integration test verifies container network isolation.** Application guards are defense-in-depth; container egress remains outer boundary.
- **DNS rebinding / Chromium DNS TOCTOU** can occur after preflight.
- **Redirects** may reach targets not covered by initial validation in unrestricted fetch paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
