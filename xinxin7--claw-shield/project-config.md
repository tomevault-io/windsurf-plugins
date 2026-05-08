---
trigger: always_on
description: > This file is written for you, the AI agent. It describes the Claw Shield project structure, conventions, and how to make effective changes.
---

# AGENTS.md — For AI Agents Working on This Codebase

> This file is written for you, the AI agent. It describes the Claw Shield project structure, conventions, and how to make effective changes.

## What Is Claw Shield

Claw Shield is an AI agent governance infrastructure. It sits between agents and model providers, providing:

1. **Privacy routing** via OHTTP (Oblivious HTTP) relay-gateway architecture
2. **Real-time telemetry** — captures chain-of-thought reasoning, tool call decisions, and execution results
3. **Model-as-a-Judge** — automated audit of high-risk tool calls via a second model, with allow/deny verdicts
4. **A hosted dashboard** for visualizing agent behavior as a reasoning-to-action waterfall, including Judge audit results

## Architecture (Three Components)

### `client/` — OpenClaw Plugin (TypeScript)

- Entry point: `client/index.ts` → registers `ClawShieldPlugin`
- Core logic: `client/src/ohttp-shield.plugin.ts`
- Intercepts outbound model requests, wraps in OHTTP, injects `x-claw-shield-project-id` and `x-claw-shield-session-id` headers
- Session ID rotates on new user prompts; persists across tool-result continuations within one agent turn
- Project ID is auto-generated UUID, persisted at `~/.openclaw/plugins/claw-shield/.project-id`
- Dependencies: `bhttp-js`, `ohttp-js`

### `relay/` — Cloudflare Worker (JavaScript)

- Simple pass-through relay (`relay/index.js`)
- Sees the client's IP but never decrypts the OHTTP payload
- Deployed via `wrangler deploy` from `relay/`

### `gateway/` — Cloudflare Worker (Rust/WASM)

- Entry: `gateway/src/lib.rs` — OHTTP decryption, request routing, Judge integration, telemetry orchestration
- Telemetry: `gateway/src/telemetry.rs` — SSE parsing for OpenAI/Anthropic/Gemini, CoT extraction, tool call logging, sensitivity detection, D1 storage
- Judge: `gateway/src/judge.rs` — Model-as-a-Judge logic: `should_invoke_judge()`, `invoke_judge()`, `build_intervention_response()`, provider-specific API calls
- Skills: `gateway/src/skills/judge_audit.md` — the Judge's system prompt (evaluation criteria), loaded via `include_str!("skills/judge_audit.md")`
- Dashboard: `gateway/src/dashboard.html` — single-page app embedded via `include_str!()`, served at `/dashboard`
- D1 database `TELEMETRY_DB` stores trace records in a `traces` table indexed by `(project_id, timestamp_ms)`, with `judge_verdict` (JSON) and `judge_action` (TEXT) columns for Judge audit results
- Migrations live in `gateway/migrations/` — apply with `npx wrangler d1 migrations apply claw-shield-telemetry --remote`
- KV namespace `WAITLIST` is used only for the waitlist feature (low-volume, read-heavy)
- API endpoints: `GET /api/traces?project=X`, `GET /api/summary?project=X` (summary includes `judge_triggered`, `judge_allowed`, `judge_denied`)
- Built with `worker-build --release`, deployed via `wrangler deploy` from `gateway/`

## Key Conventions

- **Language**: All code comments, UI text, and documentation are in English
- **Dashboard styling**: CSS custom properties defined in `:root` of `dashboard.html`. Design is dark-mode with Inter + JetBrains Mono fonts
- **Telemetry data model**: `TraceRecord` in `telemetry.rs` is the canonical schema — any new fields must be added there, to the D1 migration, and to `store_trace` / `list_traces`
- **Provider SSE parsing**: Each provider (OpenAI, Anthropic, Google) has its own `parse_*_sse` and `parse_*_json` function in `telemetry.rs`
- **Sensitivity detection**: `detect_sensitivity()` in `telemetry.rs` flags dangerous tool names (`SENSITIVE_TOOL_NAMES`) and content patterns (`SENSITIVE_CONTENT_PATTERNS`). Only explicit matches trigger the Judge — there is no catch-all keyword matching
- **Judge model selection**: `judge_config_for_provider()` in `judge.rs` maps each provider to its Judge model (e.g., OpenAI → `gpt-4.5`, Anthropic → `claude-sonnet-4-6`, Google → `gemini-3-flash-preview`). The Judge reuses the user's API key
- **Judge verdict data**: `JudgeVerdictRecord` in `telemetry.rs` stores action (allow/deny), reasoning, risk level, model, and latency. Stored as JSON in the `judge_verdict` column
- **Fail-open policy**: If the Judge invocation fails (API error, timeout), the request passes through — no silent blocking
- **Skills system**: `gateway/src/skills/` contains Markdown files defining evaluation criteria. Currently only `judge_audit.md`. These are embedded at compile time via `include_str!()`
- **No client-side telemetry storage**: All telemetry lives on the gateway D1 database. The client only injects identity headers

## How to Modify Judge Behavior

### Update evaluation criteria

Edit `gateway/src/skills/judge_audit.md`. This file is the Judge's system prompt — it defines the evaluation criteria, decision rules, and output format. Changes take effect on the next `wrangler deploy` (the file is compiled in via `include_str!`).

### Add a new sensitive tool or pattern

1. To flag a tool name: add it to `SENSITIVE_TOOL_NAMES` in `telemetry.rs`
2. To flag content patterns (arguments): add a regex-style substring to `SENSITIVE_CONTENT_PATTERNS` in `telemetry.rs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xinxin7/claw-shield](https://github.com/xinxin7/claw-shield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
