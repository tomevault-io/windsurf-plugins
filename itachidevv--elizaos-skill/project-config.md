---
trigger: always_on
description: >
---


# ElizaOS Expert Skill

## Question Mode

If this skill was invoked with an argument (e.g., `/elizaos how do I create a service?` or `/elizaos what changed in v2?`), treat the argument as a **question to answer** using the knowledge in this skill and its reference files. Answer the question directly and concisely — do NOT modify any code unless the user's original message also asked for code changes. Read the relevant reference files as needed to provide accurate answers.

## Architecture Overview

ElizaOS is a TypeScript framework (Node.js v23+, Bun) for autonomous AI agents. Core architecture:

```
ElizaOS (multi-agent manager, extends EventTarget)
  └── AgentRuntime instances (implements IAgentRuntime extends IDatabaseAdapter)
       ├── Character     — personality, knowledge, style
       ├── Plugins       — modular capability bundles
       │    ├── Actions     — what agents DO (VERB_NOUN)
       │    ├── Providers   — what agents SEE (context injection)
       │    ├── Evaluators  — what agents LEARN (post-processing)
       │    └── Services    — what agents CONNECT to (singletons)
       ├── Memory        — persistent vector DB (5 types)
       ├── Events        — 30+ async event types
       ├── Models        — provider-agnostic with priority routing
       └── Database      — Drizzle ORM + PostgreSQL/PGLite
```

### Message Processing Pipeline

```
Message In → Store in Memory → Compose State (all Providers) → shouldRespond Decision
  → LLM generates thought + actions + response → Validate & Execute Actions
  → Run Evaluators → Store Response → Deliver via Client
```

Two processing modes: **Single-Shot** (one LLM call) and **Multi-Step** (iterative with accumulated context).

### Branches & Versions (verified 2026-04-28)

| Branch | package.json | Status | Notes |
|--------|---------|--------|---------|
| `develop` (default) | `2.0.0-alpha.176` | Active alpha — this is the v2 line | 21 packages, 39 plugins, polyglot (TS+Python+Rust) |
| `main` | `1.4.4` | Legacy v1, frozen | TypeScript-only, 17 packages |
| `v2-develop` | `1.4.4` | **Stale** — identical to `main`, 3956 commits behind develop | Don't use; migrated into `develop` |
| `v2.0.0` | `2.0.0-alpha` | Separate experimental branch | Different package set incl. `computeruse/`, `sweagent/`, daemon-style packages |

**Latest published tag:** `v2.0.0-alpha.442` (2026-04-27). New alpha tags ship from `develop` continuously — install via `bun i -g @elizaos/cli@alpha`. There is **no stable v2 release** yet — all v2 work has been alpha for 13+ months.

**Recommendation:** For new work target the **`develop` branch (v2 alpha)** — it's where all development happens. Only fall back to `main` (1.4.4) if you need a frozen TypeScript-only baseline; upstream development of v1 has stopped.

### Monorepo Layout — `develop` branch (verified)

```
packages/
  agent/              → Agent runtime composition layer
  app/                → Tauri desktop wrapper
  app-core/           → Shared app logic
  benchmarks/         → Perf benchmarks
  docs/               → Mintlify docs source (docs.elizaos.ai)
  elizaos/            → CLI (npm: @elizaos/cli)
  examples/           → Example agents
  homepage/           → elizaos.ai marketing site
  interop/            → Cross-language plugin interop layer
  native-plugins/     → Built-in plugins (sql, bootstrap, etc.)
  prompts/            → Standalone prompt templates
  python/             → Python runtime + SDK (pyproject.toml, uv-managed)
  rust/               → Rust runtime + SDK (Cargo.toml, builds to native + WASM)
  scenario-runner/    → Scenario testing harness
  scenario-schema/    → Scenario format schema
  schemas/            → Protobuf schemas (buf.yaml; eliza/v1/*.proto)
  shared/             → Cross-package shared utilities
  skills/             → Reusable agent skill library
  templates/          → Project templates for `elizaos create`
  typescript/         → Core TypeScript SDK (npm: @elizaos/core)
  ui/                 → React web dashboard

plugins/              → 39 first-party plugins at top level (Discord,
                        Telegram, Twitter, Solana, EVM, OpenAI, Anthropic,
                        Ollama, OpenRouter, Google, knowledge, MCP, etc.)
```

**Key facts to internalize:**
- **Python and Rust SDKs are real and shipping** on `develop` — `packages/python/elizaos/` (pyproject + uv lock) and `packages/rust/src/` (Cargo + WASM build via `build-wasm.sh`).
- Plugins are **inside** the monorepo at top-level `plugins/`, not in a separate `elizaos-plugins` org. The old separate-org claim is obsolete.
- Capability tiers (Basic/Extended/Autonomy) are mirrored across all 3 SDKs — `AutonomyService` exists in TS, Python, and Rust (verified via `ENABLE_AUTONOMY` symbol search).

### `v2.0.0` Branch (Separate Experimental)

Distinct from `develop`. Layout:
```
packages/  @schemas/ computeruse/ daemon/ docs/ elizaos/ interop/
           milaidy/ mldy/ prompts/ psyop/ python/ rust/ samantha/
           skills/ sweagent/ tui/ typescript/
```

Adds `computeruse/` (computer-use capabilities), `sweagent/` (SWE-Agent integration), `tui/` (terminal UI), and several named character/agent packages (`milaidy`, `mldy`, `psyop`, `samantha`). Use only if you specifically want to track this branch.

### Key v2 Capabilities (vs legacy v1.4.4 on `main`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ItachiDevv/elizaos-skill](https://github.com/ItachiDevv/elizaos-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
