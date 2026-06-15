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

| Concept                  | File                                |
| ------------------------ | ----------------------------------- |
| Conductor classification | `src/conductor-bootstrap.js`        |
| Base templates           | `cluster-templates/base-templates/` |
| Message bus              | `src/message-bus.js`                |
| Ledger (SQLite)          | `src/ledger.js`                     |
| Guidance topics          | `src/guidance-topics.js`            |
| Guidance mailbox helper  | `src/ledger.js`                     |
| Guidance live injection  | `src/orchestrator.js`               |
| Trigger evaluation       | `src/logic-engine.js`               |
| Agent wrapper            | `src/agent-wrapper.js`              |
| Providers registry       | `src/providers/index.js`            |
| Provider implementations | `src/providers/`                    |
| Provider detection       | `lib/provider-detection.js`         |
| Provider capabilities    | `src/providers/capabilities.js`     |
| Start-cluster helper     | `lib/start-cluster.js`              |
| Docker mounts/env        | `lib/docker-config.js`              |
| Container lifecycle      | `src/isolation-manager.js`          |
| Settings                 | `lib/settings.js`                   |

The TUI is not included in this release. Use `zeroshot list`, `zeroshot status <id>`,
and `zeroshot logs <id> -f` or `zeroshot logs <id> -w` for monitoring.

## CLI Quick Reference

```bash
# Flag cascade: --ship -> --pr -> --worktree
zeroshot run 123                  # Local, no isolation
zeroshot run 123 --worktree       # Git worktree isolation
zeroshot run 123 --pr             # Worktree + create PR
zeroshot run 123 --pr --pr-base dev # PR base: dev, worktree base: origin/dev (incl. -d)
zeroshot run 123 --ship           # Worktree + PR + auto-merge
zeroshot run 123 --docker         # Docker container isolation
zeroshot run 123 -d               # Background (daemon) mode

# Management
zeroshot list                     # All clusters (--json)
zeroshot status <id>              # Cluster details
zeroshot logs <id> [-f|-w]        # Stream logs
zeroshot resume <id> [prompt]     # Resume failed cluster
zeroshot stop <id>                # Graceful stop
zeroshot kill <id>                # Force kill

# Utilities
zeroshot export <id>              # Export conversation
zeroshot agents list              # Available agents
zeroshot settings                 # View/modify settings
zeroshot providers                # Provider status and defaults
```

UX modes:

- Foreground (`zeroshot run`): streams logs, Ctrl+C stops cluster.
- Daemon (`-d`): background, Ctrl+C detaches.
- Attach (`zeroshot attach`): connect to daemon, Ctrl+C detaches only.

Settings: `defaultProvider`, `providerSettings` (claude/codex/gemini), legacy `maxModel`, `defaultConfig`, `logLevel`, robustness (`maxRetries`, `backoffBaseMs`, `backoffMaxMs`, `jitterFactor`, `maxRestartAttempts`, `maxTotalRestarts`, `staleWarningsBeforeKill`).

## Architecture

Pub/sub message bus + SQLite ledger. Agents subscribe to topics, execute on trigger match, publish results.

```
Agent A -> publish() -> SQLite Ledger -> LogicEngine -> trigger match -> Agent B executes
```

### Core Primitives

| Primitive    | Purpose                                                     |
| ------------ | ----------------------------------------------------------- |
| Topic        | Named message channel (`ISSUE_OPENED`, `VALIDATION_RESULT`) |
| Trigger      | Condition to wake agent (`{ topic, action, logic }`)        |
| Logic Script | JS predicate for complex conditions                         |
| Hook         | Post-task action (publish message, execute command)         |

Restart persistence: orchestrator publishes `AGENT_RESTART_ATTEMPT` to the ledger so restart limits survive orchestrator restarts.

### Guidance Messaging

- Topics: `USER_GUIDANCE_CLUSTER`, `USER_GUIDANCE_AGENT` (see `src/guidance-topics.js`).
- Mailbox helper: `ledger.queryGuidanceMailbox()` with `messageBus.queryGuidanceMailbox()` passthrough.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-open-engine/zeroshot](https://github.com/the-open-engine/zeroshot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
