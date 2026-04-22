---
trigger: always_on
description: UPDATE THIS FILE when making architectural changes, adding patterns, or changing conventions.
---

UPDATE THIS FILE when making architectural changes, adding patterns, or changing conventions.

# Zeroshot: Multi-Agent Coordination Engine

Operational rules and references for automated agents working on this repo. Install:
`npm i -g @covibes/zeroshot` or `npm link` (dev).

## CRITICAL RULES

- Never spawn without permission. Do not run `zeroshot run <id>` unless the user explicitly asks to run it.
- Never use git in validator prompts. Validate files directly.
- Never ask questions. Agents run non-interactively; make autonomous decisions.
- Never edit `CLAUDE.md` unless explicitly asked to update docs.
- Detached (`-d`) runs must forward all `zeroshot run` options via `ZEROSHOT_RUN_OPTIONS` (see `buildDaemonEnv` + `buildStartOptions`) so PR/worktree config cannot be dropped.

Worker git operations are allowed only with isolation (`--worktree`, `--docker`, `--pr`, `--ship`). They are forbidden without isolation.

Read-only safe commands: `zeroshot list`, `zeroshot status`, `zeroshot logs`

Destructive commands (need permission): `zeroshot kill`, `zeroshot clear`, `zeroshot purge`

## Where to Look

| Concept                        | File                                                       |
| ------------------------------ | ---------------------------------------------------------- |
| Conductor classification       | `src/conductor-bootstrap.js`                               |
| Base templates                 | `cluster-templates/base-templates/`                        |
| Message bus                    | `src/message-bus.js`                                       |
| Ledger (SQLite)                | `src/ledger.js`                                            |
| Guidance topics                | `src/guidance-topics.js`                                   |
| Guidance mailbox helper        | `src/ledger.js`                                            |
| Guidance live injection        | `src/orchestrator.js`                                      |
| Trigger evaluation             | `src/logic-engine.js`                                      |
| Agent wrapper                  | `src/agent-wrapper.js`                                     |
| Providers registry             | `src/providers/index.js`                                   |
| Provider implementations       | `src/providers/`                                           |
| Provider detection             | `lib/provider-detection.js`                                |
| Provider capabilities          | `src/providers/capabilities.js`                            |
| TUI backend entrypoint         | `src/tui-backend/index.ts`                                 |
| TUI backend server             | `src/tui-backend/server.ts`                                |
| TUI backend services           | `src/tui-backend/services/`                                |
| TUI backend subscriptions      | `src/tui-backend/subscriptions/`                           |
| TUI backend build output       | `lib/tui-backend/`                                         |
| TUI launcher (Node)            | `lib/tui-launcher.js`                                      |
| TUI binary mapping             | `lib/tui-binary.js`                                        |
| TUI start-cluster helper       | `lib/start-cluster.js`                                     |
| TUI binary installer           | `scripts/install-tui-binary.js`                            |
| TUI v2 protocol spec           | `docs/tui-v2/protocol.md`                                  |
| TUI v2 protocol types (TS)     | `src/tui-backend/protocol/`                                |
| TUI v2 protocol types (Rust)   | `tui-rs/crates/zeroshot-tui/src/protocol/`                 |
| Rust TUI backend client        | `tui-rs/crates/zeroshot-tui/src/backend/`                  |
| Rust TUI entrypoint            | `tui-rs/crates/zeroshot-tui/src/main.rs`                   |
| Rust TUI core loop (MVU)       | `tui-rs/crates/zeroshot-tui/src/app/mod.rs`                |
| Rust TUI spine completion      | `tui-rs/crates/zeroshot-tui/src/app/spine_completion.rs`   |
| Rust TUI input routing         | `tui-rs/crates/zeroshot-tui/src/input.rs`                  |
| Rust TUI commands              | `tui-rs/crates/zeroshot-tui/src/commands/`                 |
| Rust TUI command parser        | `tui-rs/crates/zeroshot-tui/src/commands/parser.rs`        |
| Rust TUI command dispatch      | `tui-rs/crates/zeroshot-tui/src/commands/dispatcher.rs`    |
| Rust TUI command types         | `tui-rs/crates/zeroshot-tui/src/commands/types.rs`         |
| Rust TUI screens               | `tui-rs/crates/zeroshot-tui/src/screens/`                  |
| Rust TUI Fleet Radar screen    | `tui-rs/crates/zeroshot-tui/src/screens/radar.rs`          |
| Rust TUI Cluster Canvas screen | `tui-rs/crates/zeroshot-tui/src/screens/cluster_canvas.rs` |
| Rust TUI render entrypoint     | `tui-rs/crates/zeroshot-tui/src/ui/mod.rs`                 |
| Rust TUI widgets               | `tui-rs/crates/zeroshot-tui/src/ui/widgets/`               |
| Rust TUI toast widget          | `tui-rs/crates/zeroshot-tui/src/ui/widgets/toast.rs`       |
| Rust TUI command bar widget    | `tui-rs/crates/zeroshot-tui/src/ui/widgets/command_bar.rs` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [covibes/zeroshot](https://github.com/covibes/zeroshot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
