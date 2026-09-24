---
trigger: always_on
description: A native Android app that runs the OpenClaw AI Gateway locally on the phone and
---

# 4AIs Android — AI Agent on Your Phone

## What this is
A native Android app that runs the OpenClaw AI Gateway locally on the phone and
lets an LLM control the device through an AccessibilityService. 100%
bring-your-own-key: the user picks NVIDIA (free tier), OpenAI, Anthropic, or
Google Gemini in Settings and the app talks to that API directly. There is no
backend server, no accounts, no telemetry.

## Architecture
```
Compose UI ─► WsRpcClient ──ws://127.0.0.1:3000──► OpenClaw Gateway (Node.js on-device) ──► user's provider
                  │                                                                          (NVIDIA/OpenAI/
                  └─► ApprovalChannel (exec/plugin.approval.requested → risk-graded dialog)   Anthropic/Gemini)
                       ChatSession ─► method:"agent" ─► event:"agent" {stream: assistant|tool|lifecycle}
                                                                                ▲
                        agent runs `act …` via the exec tool ───────────────────┘
                                     │
                    filesDir/bin/act ─► http://127.0.0.1:3001 (ShizukuBridge, bearer auth)
                                            ├─ /agent/act, /ui/* ─► UiRoutes ─► AccessibilityService
                                            └─ /exec ─► Shizuku shell (optional)
```

- **OpenClaw is WebSocket-RPC, NOT OpenAI REST.** `/v1/chat/completions` does
  not exist on openclaw 2026.5.12. The client speaks JSON-RPC frames
  (`{type:"req", id, method, params}` ↔ `{type:"res", id, ok, payload|error}` +
  unsolicited `{type:"event", event, payload}`) over one long-lived WebSocket.
- **Connect handshake.** Server emits `connect.challenge`; client sends
  `method:"connect"` with `role:"operator"`, `client.mode:"backend"` (loopback
  shortcut, no device crypto), and the persistent token from
  `~/.openclaw/auth-token`; server returns hello-ok.
- **Chat is the `agent` method.** Streaming arrives as `event:"agent"` frames
  (payload fields nested under `data`), demuxed by `ChatSession` into a sealed
  `ChunkEvent` (`TextDelta` / `ToolStart` / `ToolResult` / `Lifecycle` /
  `Usage` / `AgentError` / `Done`). Every turn uses a fresh `sessionKey` (token
  economy — openclaw would otherwise replay all prior tool results);
  conversational memory is re-added as a text-only prelude (6 turns × 240
  chars). Runaway guard kills any turn crossing 250k input tokens.
- **Providers are BYO-key and rebuilt every start.** `ProviderCatalog`
  (`data/providers/`) defines nim/oai/ant/gem as CUSTOM openai-completions
  providers (explicit baseUrl + models + `agentRuntime: pi`). NEVER use
  openclaw's built-in provider plugins ("nvidia", "openai", …) — their static
  model catalogs are stale and reject valid models. Keys live in
  `EncryptedKeyStore`; `NodeProcess.refreshMcpConfig` rewrites
  `models.providers` + `defaults.model.primary` from keystore + prefs on every
  gateway start. config.json is a derived artifact — hand edits won't stick.
- **Tool calls use the `act` command → UiRoutes on :3001.** openclaw 2026.5.12
  dropped stdio MCP transport, so the agent's only built-in tool is `exec`.
  `NodeProcess.writeActWrapper` installs an `act` shell command into
  `filesDir/bin` (on PATH, rewritten every gateway start so APK updates
  redeploy it). Verbs: observe/look/tap/double_tap/swipe/type/wait/back/home/
  note/take_over/launch/batch. The wrapper injects the bearer token and POSTs
  to `/agent/act`, `/ui/app/launch`, `/ui/batch`.
  `ChatSession.phoneCapabilitySuffix` (~280 tokens, sent every turn) teaches
  the grammar and explicitly says Shizuku is optional.
- **UI automation via AccessibilityService.** `PhoneAccessibilityService` →
  `AccessibilityServiceHolder` (@Singleton) → `UiRoutes` behind `ShizukuBridge`
  (NanoHTTPD on :3001, constant-time bearer check on every route). Capabilities:
  tree with stable legend ids (`MarkAssigner`), occlusion filtering, tap by
  id/text, scroll-to-find, wait-for-element, screenshots with legend overlay,
  post-action legend diff, per-app agent notes, clipboard, notifications,
  agent-vision overlay, `take_over` manual handoff, audit log (`AgentAuditLog`).
- **Approval flow.** `ApprovalChannel` handles both `exec.approval.requested`
  and `plugin.approval.requested`, classifies risk
  (`SensitiveIntentClassifier`), and surfaces `ApprovalDialog`. Auto-approve
  (Settings, default OFF) skips low-risk prompts, but sensitive intents ALWAYS
  prompt.
- **Health is `/health` and must be JSON.** The gateway's SPA fallback returns
  200 text/html for any URL; `NodeProcess.checkHealth` requires a JSON
  content-type to avoid the false positive.
- **Bootstrap.** First launch: `node-setup.js` extracts bundled `npm.tgz`, runs
  `npm install openclaw@2026.5.12 --ignore-scripts`, generates the gateway
  token (written to config.json AND the 0600 `auth-token` sidecar). Gated by
  `.4ais_setup_done_v10` — bump `SETUP_VERSION` in node-setup.js AND the marker
  in `NodeSetup.kt` together or changes never land on existing installs.
- **Node runs from the APK.** `libnode.so` in `nativeLibraryDir` (Termux-style
  build, `TERMUX_APP__PACKAGE_NAME=com.crsk.openclaw`), executed directly via

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [8crsk/openclaw-android](https://github.com/8crsk/openclaw-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
