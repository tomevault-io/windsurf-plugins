---
trigger: always_on
description: Onboarding for coding AIs (Claude Code, Cursor, Codex, …) contributing to **YiYi**.
---

# AGENTS.md

Onboarding for coding AIs (Claude Code, Cursor, Codex, …) contributing to **YiYi**.

If you're a human reader, this also works as a fast tour — just skim §1 then jump to whatever section matches the task.

---

## 1. 30-second tour

YiYi is a **desktop AI personal assistant** — Tauri (Rust backend) + React 18 / TypeScript frontend. Single binary, runs locally on the user's machine. Data lives in SQLite (WAL mode) under `~/.yiyi/`.

Differentiation vs. other open-source agents:
- **Deeply tuned for DeepSeek V4** (Pro / Flash) — implicit prefix cache, 1M context, 120× hit/miss price gap drives most architectural choices.
- **White-box co-construction** — the agent never silently rewrites its own skills or rules. Anything it proposes (new skill, new principle) goes through an *Inbox* the user approves. See §6.
- **Multi-platform bots** out of the box — Discord, QQ, Telegram, DingTalk, Feishu, WeCom, Webhook.

**Default behaviour expectations:**
- Read code before editing. Keep changes scoped. No speculative abstractions.
- Don't break the prefix cache (see §7).
- Don't add features the user didn't ask for.
- Finish things; "能跑 ≠ 能用，能用 ≠ 好用".

---

## 2. Repo layout

```
/
├── AGENTS.md           ← this file (you're here)
├── README.md           ← public-facing intro / screenshots
├── app/
│   ├── src/            ← React frontend (Vite + TS + Tailwind)
│   │   ├── pages/      Chat, CronJobs, Bots, Skills, Settings, …
│   │   ├── components/
│   │   ├── api/        Tauri invoke wrappers
│   │   ├── stores/     Zustand stores
│   │   └── i18n.ts     zh/en strings
│   └── src-tauri/      ← Rust backend (the heart of YiYi)
│       ├── src/
│       │   ├── lib.rs          App init + plugin setup
│       │   ├── main.rs         CLI dispatch (`yiyi doctor`, …) → lib.rs::run()
│       │   ├── doctor.rs       Environment self-check
│       │   ├── tray.rs         System tray menu
│       │   ├── commands/       Tauri command handlers
│       │   │   ├── agent/      chat / streaming / history
│       │   │   ├── bots.rs / cronjobs.rs / skills.rs / workspace.rs / …
│       │   ├── engine/         ← Core domain logic
│       │   │   ├── react_agent/
│       │   │   │   ├── core.rs         Think → Act → Observe loop
│       │   │   │   ├── prompt.rs       System prompt + persona prefix
│       │   │   │   ├── compaction.rs   Long-context handling
│       │   │   │   ├── growth.rs       Skill / principle proposals → Inbox
│       │   │   │   └── loop_guard.rs   Anti-loop guard per turn
│       │   │   ├── tools/      ← Tool dispatch (heavy file, expect 3k+ LOC)
│       │   │   │   ├── file_tools.rs   read/write/edit/append/delete + undo_edit
│       │   │   │   ├── shell_security.rs  Two-layer regex (Hardline ▸ Block ▸ Warn)
│       │   │   │   ├── output_envelope.rs  Trust + MultimodalEnvelope
│       │   │   │   ├── permission_gate.rs  Native OS approval dialog
│       │   │   │   └── …
│       │   │   ├── llm_client/         OpenAI / Anthropic / Google formats
│       │   │   ├── bots/               Per-platform adapters + BotManager
│       │   │   ├── infra/              mcp_runtime, dep_check, python_bridge
│       │   │   ├── db/                 SQLite schema + queries
│       │   │   ├── checkpoint.rs       Shadow-git per-turn snapshots
│       │   │   ├── scheduler.rs        Cron / delay / once jobs
│       │   │   ├── prompt_cache.rs     FNV fingerprint + cache-break detect
│       │   │   └── tool_registry_global.rs  Single source of truth for tools
│       │   ├── state/                  AppState, Config
│       │   └── test_support/           Test helpers (feature-gated)
│       ├── tests/      ← Integration tests (flat layout, one file per area)
│       └── Cargo.toml
└── evals/              ← Behaviour eval YAML cases + rubric
```

**Heuristic for "where do I put this?"**:
- *New tool* → `engine/tools/<area>_tools.rs` + register via `tool_registry_global`.
- *New Tauri command* (frontend can call) → `commands/<area>.rs`.
- *New DB table* → `engine/db/<table>.rs` + add to `engine/db/mod.rs`.
- *New frontend page* → `app/src/pages/X.tsx` + register in routes.
- *Skill content* → `app/src-tauri/skills/<name>/SKILL.md` (built-in) or document the path for `~/.yiyi/active_skills/`.

---

## 3. Commands

### Day-to-day
```bash
# from app/
npm run tauri dev           # full app, hot-reload frontend, rebuild Rust on change
npm run dev                 # frontend only (Vite)
npx tsc --noEmit            # frontend type-check (no emit)

# from app/src-tauri/
cargo check --features test-support       # fast type check
cargo test  --features test-support       # default (hermetic) test tier
cargo test  --features test-support,test-integration  # +live API tier
cargo build                                # debug build for `target/debug/yiyi`
```

### After a build, sanity-check the environment
```bash
./app/src-tauri/target/debug/yiyi doctor   # 10 checks in <1s; exit code = fail count
```

### Test conventions
- New integration tests live at `tests/<area>.rs` (flat — no sub-dirs).
- Async default: `#[tokio::test(flavor = "multi_thread")]`.
- Anything touching SQLite uses `TempDb` + `#[serial]` (WAL can't be shared across parallel threads).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibeinging/YiYi](https://github.com/vibeinging/YiYi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
