---
trigger: always_on
description: Tail runtime logs from any deployment platform (Vercel / file / stdin / custom command), group by a configurable request-id regex, classify per a project-supplied known-bug ruleset, dedupe via signature hash, surface NEW bugs as macOS notifications + auto-filed `.agent/issues/auto-NNNN-*.md`, write lighter case + anomaly records for known-bug hits + out-of-scope findings into `.agent/issues/case-NNNN-*.md` and `.agent/issues/anomaly-NNNN-*.md`, and append to `.agent/monitor-rollup.md`. Auto-INDE
---


# /monitor-deploy — generic runtime-log watcher

> **Project-agnostic.** Originally written for a Vercel codex-pdf deploy; refactored so any project can drop it in, point it at their runtime logs, supply a request-id regex + a seed of known signatures, and get the same auto-classify / dedupe / surface loop. See `monitor-deploy.config.example.yaml` for the full config schema.

The skill watches runtime logs autonomously, classifies failures per a known-bug ruleset, and surfaces NEW signatures the moment they appear — so deploy bugs go from "user reports broken canary" → "issue auto-filed for `/afk-agents`" without a human in the loop.

## When to invoke

- One-shot: `/monitor-deploy` — does one polling tick and exits
- Continuous: `/loop 5m /monitor-deploy` — fires every 5 min. Cheap to no-op; most ticks find nothing new.
- The skill is **idempotent**: re-running with the same state log + the same logs produces zero new alerts.

## Inputs (optional positional args)

- **First arg**: log source override. If unset, uses `log_source` from `monitor-deploy.config.yaml` (resolved relative to the cwd).
- **Second arg**: lookback window (e.g. `15m`, `1h`). Default = `since last_seen_ts in .agent/monitor-state.json` OR `default_lookback` from config (default `15m`) on first invocation.

## Configuration

The skill reads `monitor-deploy.config.yaml` from the cwd. **All project-specific knobs live in the config**, not in this file. See `monitor-deploy.config.example.yaml` for the full schema with sensible defaults.

Minimal example for a Vercel project:

```yaml
log_source:
  type: vercel
  project_id: null              # falls back to .vercel/project.json:projectId
  team_id: null                 # falls back to .vercel/project.json:orgId
  environment: production
  limit: 100
reqid_pattern: '\[(job|api|worker) ([a-z0-9]{6})\]'
slow_threshold_ms: 30000
default_lookback: 15m
notification:
  enabled: true
  title: "<project-slug>: new bug signature"
  sound: Funk
issues_dir: .agent/issues
state_file: .agent/monitor-state.json
rollup_file: .agent/monitor-rollup.md
rollup_max_lines: 500
```

Minimal example for a self-hosted Docker / k8s project using `kubectl logs`:

```yaml
log_source:
  type: command
  cmd: "kubectl logs -n prod -l app=api --since=<lookback> --tail=200"
  # The skill will append `--since=<lookback>` if your cmd includes the literal <lookback>
reqid_pattern: '\[([a-z\-]+) ([a-z0-9]{6,12})\]'
```

Minimal example for a project whose logs land on disk:

```yaml
log_source:
  type: file_tail
  paths: ["./logs/api.log", "./logs/worker.log"]
  # Skill tracks per-file byte offsets in state; new lines per tick are processed.
```

If no config exists, the skill fails fast with a one-liner pointing to `monitor-deploy.config.example.yaml` — it does NOT silently fall back to Vercel defaults.

## What the skill does, top to bottom

1. **Read state** from `<state_file>` (default `.agent/monitor-state.json`):
   ```json
   { "last_seen_ts": "2026-06-13T01:35:00Z", "tick_count": 12, "total_alarms": 0, "new_signatures_this_session": 0 }
   ```
   If the file doesn't exist, treat `last_seen_ts` as `default_lookback` ago and create the file at the end.

2. **Read known signatures** from `.agent/bug-signatures.json`:
   ```json
   {
     "sig:<your-slug>": {
       "first_seen": "2026-06-12T11:00:00Z",
       "hit_count": 9,
       "pattern": "<known-bad substring or regex>",
       "memo": "<human-readable explanation + suggested fix>"
     }
   }
   ```
   **This file is project-owned.** The skill seeds it with whatever the project supplies; it does NOT bundle defaults. The 4 seed examples that ship in this skill's `examples/bug-signatures.example.json` are for a Vercel PDF render — copy them as a starting point or replace with your own.

3. **Pull runtime logs** via the configured `log_source`:
   - `type: vercel` → call `mcp__plugin_vercel_vercel__get_runtime_logs` (needs the Vercel MCP loaded; falls back to "platform unavailable" anomaly if MCP absent)
   - `type: file_tail` → `tail -n 200 <paths> | sort -k1,1` (offset persisted in state for incremental reads)
   - `type: command` → `bash -lc "<cmd>"`; if the cmd string contains the literal `<lookback>`, the skill substitutes the resolved window
   - `type: stdin` → read all of stdin (for piping from a curl in a shell pipeline)
   - Don't pass a `level` filter — the classifier needs info-level lines too, so it can group by reqId.

4. **Group lines by reqId.** For each line, apply the configured `reqid_pattern` (default `\[(job|api|worker) ([a-z0-9]{6})\]`). Bucket every line whose reqId matches into one job. Lines without a reqId (raw tracebacks, request-summary lines) go into a `_unbound` bucket — these are surfaced as anomalies, not as classified jobs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liush2yuxjtu/monitor-deploy](https://github.com/liush2yuxjtu/monitor-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
