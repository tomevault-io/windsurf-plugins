---
trigger: always_on
description: ├── proxy.js              # Main proxy implementation (1646 lines)
---

# FREEBUFF-PROXY Development Guide

## Project Structure

```
FREEBUFF-PROXY/
├── proxy.js              # Main proxy implementation (1646 lines)
├── dashboard.html        # Liquid glass dashboard with OAuth UI (1023 lines)
├── .config/
│   └── config.json       # Runtime configuration
├── package.json          # Project metadata (freebuff, node-forge, node-fetch, socks-proxy-agent)
├── start.cmd             # Auto-detect launcher (Bun preferred, Node fallback)
├── start-node.cmd        # Node.js-only launcher
├── README.md             # User documentation
└── AGENTS.md             # This file (developer guide)
```

## Key Components

### 1. Constants & Version Tracking (lines 1-99)

- Source URLs for GitHub TypeScript files and Rust reference
- Version constants: `BUN_VERSION`, `FREEBUFF_CLI_VERSION`, `AI_SDK_COMPAT_VERSION`
- `CANONICAL_MODEL_ALIASES` — Maps shorthand model names to full IDs (e.g. `deepseek-v4-pro` → `deepseek/deepseek-v4-pro`)
- `FALLBACK_AGENT_IDS` — Hardcoded model-to-agent mapping when registry unavailable
- `checkAndUpdateVersions()` — Fetches `freebuff2api_rs` source and npm registry to auto-update version strings
- User-Agent generators: `getApiUserAgent()`, `getChatUserAgent()`, `getAdsUserAgent()`

### 2. Config System (lines 101-176)

- `loadConfig()` — Loads `.config/config.json` with env var overrides (`LISTEN_ADDR`, `UPSTREAM_BASE_URL`, `REQUEST_TIMEOUT`, `AUTH_TOKENS`, `API_KEYS`)
- `loadFreebuffCLITokens()` — Reads `~/.config/manicode/credentials.json`, extracts all `authToken` entries
- `saveConfig()` — Writes current config back to `.config/config.json`
- `parseDuration()` — Parses duration strings like `15m`, `6h`, `30s`
- Auto-normalizes `codebuff.com` → `www.codebuff.com`

### 3. ModelRegistry (lines 178-280)

- `start()` — Fetches models immediately, then refreshes every 6 hours
- `refresh()` — Parallel fetch of `free-agents.ts` and `freebuff-models.ts` from GitHub
- `parseConstants(source)` — Regex extracts `export const X = 'value'` into a Map
- `parseAllFreeModels(source, variableMap)` — Regex extracts `'agent-id': new Set([MODEL_VAR, ...])` blocks, resolves variables
- `buildModelMapping()` — Uses hardcoded `SUPPORTED_MODELS` map (4 models → 4 agents)
- Result: `modelToAgent` Map, `allModels` array

### 4. UpstreamClient (lines 282-419)

- `doJSON(authToken, path, body, method, extraHeaders)` — Generic JSON request with AbortController timeout
- `startRun(authToken, agentID, ancestorRunIds)` — `POST /api/v1/agent-runs` with `action: 'START'`
- `finishRun(authToken, runID, totalSteps)` — `POST /api/v1/agent-runs` with `action: 'FINISH'`
- `recordRunStep(authToken, runID, stepNumber, childRunIds, messageId, startTime)` — `POST /api/v1/agent-runs/{id}/steps`
- `chatCompletions(authToken, body, proxyAgent)` — `POST /api/v1/chat/completions` (streaming-aware, uses `node-fetch` + `SocksProxyAgent` when proxyAgent provided, global `fetch` otherwise)
- `createSession(authToken, model, proxyAgent, countryCode)` — `POST /api/v1/freebuff/session`
- `getSession(authToken, instanceID, proxyAgent)` — `GET /api/v1/freebuff/session` with `x-freebuff-instance-id` header
- `endSession(authToken, instanceID)` — `DELETE /api/v1/freebuff/session`
- Handles 426 (`freebuff_update_required`) and `model_locked` errors specially

### 5. TokenPool (lines 421-567)

- Manages multiple auth tokens with round-robin selection
- Mutex-based locking via promise chain (`withLock()`)
- `ensureSession(token, model)` — Up to 3 retries, handles model_locked and freebuff_update_required
- Session data stored: `status`, `instanceID`, `expiresAt`, `countryCode`, `remainingMs`
- `pollUntilReady(token, model, state)` — Polls up to 60 iterations for `active` status, handles `queued`, `ended`, `superseded`, `disabled`
- `endAllSessionsForToken(token)` — Cleans up all sessions for a token
- `invalidateSession(token, model)` — Removes specific session from cache
- Session key format: `{token}:{model}`

### 6. WarpPlusManager (lines 412-520)

- Manages a SOCKS5 proxy via the `warp-plus` binary for bypassing rate limits
- `ensureBinary()` — Downloads `warp-plus.exe` from GitHub releases if not present
- `start()` — Spawns the binary on `127.0.0.1:8086`, waits up to 20s for readiness
- `_waitForReady(timeout)` — Polls SOCKS5 connectivity via `nodeFetch` to `api.ipify.org`, checks process is still alive
- `stop()` — Kills the process and resets state
- `isReady()` — Returns true when process is running and proxy agent is created
- `getAgent()` — Returns `SocksProxyAgent` instance for use with `node-fetch`
- `lastEndpoint` — Caches the last working WARP endpoint (IP:port) for reuse on restart
- Used by `proxyChatRequest` when `accessTier === 'limited'` to route through Cloudflare WARP

### 7. Run Chain Helpers (lines 569-603)

Two distinct run chain patterns:

**Normal chain** (`startRunChainNormal`):
1. Start parent run (e.g. `base2-free`)
2. Start child run (`context-pruner`) with parent as ancestor
3. Record step + finish child run
4. Record step on parent with child run ID

**Gemini chain** (`startRunChainGemini`):
1. Start parent run
2. Start chat run with parent as ancestor

Finalization:
- `finalizeRunChainNormal` — Records step 2 + finishes parent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [notBlubbll/free-buff-lol](https://github.com/notBlubbll/free-buff-lol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
