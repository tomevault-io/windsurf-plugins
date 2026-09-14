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

### Deny-by-default child environments (CLI/MCP/native)
`src/cli/cli-backend.ts:buildCliEnvironment` gives every CLI child the nonsecret base config only, plus per-tool-family credentials (`CLI_TOOL_CREDENTIALS`) — a `web_search` child never carries GitHub/Reddit/graph secrets and vice versa; unknown tool names get base config only. `src/process/mcp-client.ts:toProcessEnvironment` is deny-by-default: only listed provider credentials, benign client config, and names in the explicit `SEARCH_MCP_FORWARD_ENV_JSON` allowlist forward — there is no `SEARCH_MCP_*` wildcard, and `parseForwardedEnvironmentKeys` rejects secret-like names (`SECRET_LIKE_NAME_PATTERN`) and non-benign `SEARCH_MCP_*` internals. `src/process/native-child-env.ts:buildNativeChildEnvironment` (git/ffmpeg/media CLIs) takes a minimal OS-spawn allowlist only — no tokens/keys/cookies, no proxy config, no interpreter/linker/git-config/cert overrides — with fixed argv arrays and `shell: false` (documented, test-enforced).

### Local sidecar stdin-token auth
`src/sidecar/sidecar-manager.ts:SidecarManager.start` mints a fresh 256-bit token per start (`randomBytes(32)`) and delivers it over the child's stdin pipe only (`SIDECAR_TOKEN=<hex>\n`, via `writeAuthToken`) — never argv/env (the child env is allowlisted and would strip it anyway), never logged. Delivery is mandatory: missing stdin or a write failure is a startup failure, never an unauthenticated running sidecar. The token clears on `stop()`/exit/crash; restarts mint fresh. `src/sidecar/embedding-client.ts` takes the token via explicit `apiToken` or a per-request `apiTokenProvider` (wins fresh on every request, so long-lived clients survive restarts); `EMBEDDING_SIDECAR_API_TOKEN` env fallback is for external sidecars only (`EMBEDDING_SIDECAR_BASE_URL` Bearer health check).

### Leaf-runtime RPC: fixed safe errors, reject-not-clamp, provider-opaque DTOs
`src/runtime/runtime-rpc-protocol.ts:RUNTIME_RPC_ERROR_MESSAGES` is the closed set of safe messages — `wireErrorToSafe` in `src/runtime/leaf-runtime-client.ts` maps unknown codes to `provider_error`; provider exception text never crosses. `safeLeafCode` in `src/web/agent/agent-jobs.ts` allowlists `[a-z_]{1,64}`, else `provider_error`. Out-of-range `timeoutMs`/prompt bytes/`maxOutputTokens` reject, never clamp (`asTimeoutMs`, `runLeaf`); clone ceilings are operator-lower-only (`resolveGithubClonePolicy`); the MCP forward list rejects secret-like/invalid names; vision eligibility never broadens on failure (`eligibleTiersAfterFailure`). Provider-opaque DTOs: `runLeaf` resolves to `{ text }` only (runId stays internal, metadata redacted via `redactProvenance`); job snapshots carry `transport` + safe `reason` only — never provider/model identity (`src/web/agent/agent-rpc.ts`, `negotiateLeafTransport`).

### Event-bus trust boundary
The leaf-runtime `LeafEventBus` (`src/runtime/leaf-runtime-client.ts`) is an in-process seam for trusted co-installed extension modules only (provider registered via `setLeafRuntimeProvider`, wired in `src/index.ts` when `PI_NORTHSTAR_LEAF_MODEL` is set) — it is not an authenticated channel. The `RuntimeCorrelationV1` metadata (`owner: 'northstar'`, bounded ASCII `correlationId`/`stage`, bounded `queryIndex`/`attempt`, closed role set) is routing/observability metadata with exact-keys validation, not auth: unknown fields reject, but nothing in it proves caller identity.

### Python child processes MUST use the shared env allowlist

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
