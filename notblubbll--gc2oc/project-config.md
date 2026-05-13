---
trigger: always_on
description: Ollama-emulating proxy that connects **GitHub Copilot Chat & Agent** (VS 2026 / VS Code) to the [OpenCode](https://opencode.ai) Zen + Go APIs + free models. Exposes an Ollama-compatible HTTP API on `localhost:11434` so the GitHub Copilot extension's built-in Ollama provider can use OpenCode models.
---

# AGENTS.md

## gc2oc Proxy

Ollama-emulating proxy that connects **GitHub Copilot Chat & Agent** (VS 2026 / VS Code) to the [OpenCode](https://opencode.ai) Zen + Go APIs + free models. Exposes an Ollama-compatible HTTP API on `localhost:11434` so the GitHub Copilot extension's built-in Ollama provider can use OpenCode models.

**Runtime:** Bun (preferred) → Node.js fallback. Framework: Hono. Launcher: C# `service.exe` (dual-mode: console + Windows service, embedded in `build-node.cmd`).

---

## Architecture

```
VS 2026 / VS Code                    gc2oc Proxy                 Upstream APIs
─────────────────                    ──────────────────                  ─────────────
GitHub Copilot extension             src/server.js                       OpenCode Zen (free)
  │  Ollama provider                   │  Hono HTTP server                 │  https://opencode.ai/zen/v1
  │  → /api/tags                       │  → model registry                 │
  │  → /v1/chat/completions            │  → chat completion                │
  │  → /api/chat                       │  → compression                    │  OpenCode Go (paid)
  ▼                                    │  → caching                        │  https://opencode.ai/zen/go/v1
                                    ┌───┴──────────────┐                  │
                                    │ Client detection  │                  │
                                    │ ├─ isVS2026()     │                  │  Pollinations (free)
                                    │ ├─ isVSInsiders() │                  │  https://text.pollinations.ai/openai
                                    │ ├─ isVSCode()     │                  │
                                    │ ├─ isSqlStudio()  │                  │
                                    │ └─ LocalPilot     │                  │
                                    └───┬──────────────┘                  │
                                        │                                  │  M365 Copilot (opt)
                                    ┌───┴──────────────┐                  │  ws://127.0.0.1:8765
                                    │ Model routing     │                  │  → substrate.office.com
                                    │ ├─ M365 → relay   │                  │
                                    │ ├─ Free → Zen     │                  │
                                    │ ├─ Poll → Poll.   │                  │
                                    │ └─ Paid → Go      │                  │
                                    └───┬──────────────┘                  │
                                        │                                  │
                                    ┌───┴──────────────┐                  │
                                    │ Concurrency mgr   │                  │
                                    │ ├─ thinking queue │                  │
                                    │ └─ standard queue │                  │
                                    └───┬──────────────┘                  │
                                        │                                  │
                                    ┌───┴──────────────┐                  │
                                    │ Response pipeline │                  │
                                    │ ├─ tool extraction│                  │
                                    │ ├─ think parsing  │                  │
                                    │ └─ SSE formatting │                  │
                                    └──────────────────┘                  ▼
```

---

## Source Files

### `src/server.js` (~1970 lines)
**Main entry point.** HTTP routing, request orchestration, response formatting.

- `restartSelf(exitCode)` — spawn `cmd /c start` → new console → inner `cmd /c` runs the exe. Uses `process.execPath` (NOT `process.argv[0]` — in Bun-compiled binaries `argv[0]` is `"bun"`, not the exe path). Wrapped mode (`GC2OC_WRAPPED=1`): just `process.exit()`.
- `isVSCode(c)` / `isVS2026(c)` / `isVSInsiders(c)` / `isSqlStudio(c)` — client detection from `User-Agent` (VS Code) and `baggage` header (VS/VS Insiders/SQL Studio)
- `extractVSContext()` / `getWorkspaceRoot()` / `getActiveFile()` — extract workspace paths from VS context blocks
- `extractToolCalls()` — parse AI response text into tool calls (markdown blocks → `create_file`)
- `processThinkTags()` — `<think>` tag extraction for DeepSeek reasoning
- `_simStream()` — simulated SSE streaming from non-stream responses (VS 2026)
- `_cacheReasoning()` / `_getReasoning()` — per-message reasoning cache (DeepSeek)
- `normalizeOpenAIParams()` — camelCase → snake_case parameter mapping
- `sanitizeContent()` — strip `<|im_start|>`, `<|im_end|>` tokens
- `mapModel(name)` — resolve model names, strip `[FREE]`/`[GO]`/`[M365]` prefixes

### `src/opencode-client.js` (1423 lines)
**Model registry + upstream API communication.**

- `config` — env-var-backed configuration getters
- `initModels()` — bootstrap model list (disk cache → fetch → background refresh)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [notBlubbll/gc2oc](https://github.com/notBlubbll/gc2oc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
