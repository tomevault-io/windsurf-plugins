---
trigger: always_on
description: > Operations-sensitive local options monitoring system.
---

# Agent Manual — options-monitor

> Operations-sensitive local options monitoring system.
> Treat this repo as a controlled production tool, not a sandbox.

## Repo-Specific Contract

- Treat current source, config, tests, and runtime artifacts as authority; memory and file names are only hints.
- Prefer root-cause fixes at the owning boundary. If a tactical patch is unavoidable, state the tradeoff and follow-up.
- Follow parsimony at repo boundaries: do not add entities, layers, states, tools, config keys, or workflows unless they are necessary.
- Preserve user changes in a dirty worktree. Never reset or revert unrelated files unless explicitly asked.
- Do not invoke Gateflow, planreview, or deepreview unless the user explicitly requests the named workflow or skill in the current task.

## Project Identity

| Property | Value |
|---|---|
| Purpose | Sell Put / Covered Call / Yield Enhancement scanning, filtering, reporting, and notification |
| Stack | Python 3, pandas, SQLite, OpenD/Futu API, Feishu webhooks |
| Accounts | Lowercase labels such as `lx`, `sy`; read from top-level `accounts` in runtime config |
| Canonical Configs | `config.yaml` is the human authoring source; `config.us.json` / `config.hk.json` are generated runtime snapshots |
| Reports | `output/`, `output_accounts/<account>/`, `output_shared/`, `output_runs/<run_id>/` |
| Local Tool Gateway Entry | `./om-agent` |
| Human CLI Entry | `./om` |
| Detailed Agent Handbook | `docs/AGENT_WIKI.md` |

## Entry Point Ladder

Use the highest-level safe entry point available:

1. `./om-agent` for structured JSON tools and read-first diagnostics. It is the local Tool Gateway, not OM's autonomous Agent.
2. `./om` for human/operator CLI workflows.
3. `./.venv/bin/python -m src.application.<module>` only when no public facade exists.
4. `./.venv/bin/python scripts/...` only for compatibility or operational wrappers.

Unified tick chain:

```bash
./om run tick --config config.us.json --accounts lx [sy]
```

Production cron normally uses the guarded wrapper:

```bash
./om run tick-cron --market us --accounts lx sy --timeout 600
./om run tick-cron --market hk --accounts lx sy --timeout 600
```

Legacy `scripts/send_if_needed*.py` is removed. Do not use it.

## Safety Red Lines

Ask for explicit confirmation before any command that can:

- Send real notifications through Feishu, webhook, email, or another channel.
- Install, start, stop, or modify production services such as systemd / launchd units.
- Modify `config.yaml`, `config.us.json`, `config.hk.json`, secrets, or production runtime config.
- Delete `output/`, `output_runs/`, `output_shared/`, state files, caches, or runtime artifacts.
- Write Feishu, option-position state, trade events, or broker-facing data.

When a dry-run or read-only surface exists, use it first.

## Request Defaults

| User intent | Repo-specific default |
|---|---|
| explain / look into / check / why / how does this work | Start read-only; inspect source, docs, configs, tests, and runtime artifacts before proposing changes |
| commit and push / 提交并推送 | Commit and push the named development change only; do not modify `VERSION`, publish a Release, or upgrade production unless explicitly requested |
| release / 发布 | Prepare and publish the full VERSION-driven GitHub Release; production upgrade remains a separate explicit action |
| release and upgrade / 发布并升级远端 | Publish the VERSION-driven Release, then use the controlled remote upgrade and runtime verification flow |
| diagnostic only / 不要改文件 | Keep commands read-only and do not edit files |

Do not run Python scripts just to see what happens.

## Fast Diagnostic Commands

```bash
./om-agent run --tool runtime_status --input-json '{"config_key":"us"}'
./om-agent run --tool healthcheck --input-json '{"config_key":"us"}'
./om-agent run --tool config_validate --input-json '{"config_key":"us"}'
./om-agent run --tool scheduler_status --input-json '{"config_key":"us","account":"lx"}'
```

Research evidence handoff for MacBook Codex:

```bash
./om research collect --config-key us --scope full --output both --no-write-outputs
```

## Module Ownership

| Task | Primary owner | Guardrail |
|---|---|---|
| Candidate filter/rank logic | `domain/domain/engine/candidate_engine.py` | Do not add parallel ranking in application scan adapters |
| Candidate trace / ranking diagnostics | `src/application/agent_tools/candidate_filter_impl.py`, `src/application/agent_tools/candidate_rank_impl.py` | Keep analysis read-only unless explicitly designing a write path |
| Notification text | `src/application/daily_decision_brief_renderer.py`; compatibility formatting in `src/application/notify_symbols.py`, `src/application/multi_tick/notify_format.py` | Keep Markdown-friendly Chinese text; Daily Brief owns ordinary scheduled delivery |
| Close-advice policy | `domain/domain/close_advice.py` | Runner assembles I/O; scoring policy stays in domain |
| Option-position projection | `domain/domain/ledger/projection.py` | `trade_events -> projection -> position_lots` is canonical |
| Ledger application boundary | `src/application/ledger/api.py` | Non-ledger modules must not import ledger internals directly |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuxie066/options-monitor](https://github.com/liuxie066/options-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
