---
trigger: always_on
description: Guide for AI agents and scripts running corgi non-interactively.
---

# Driving corgi as an agent

Guide for AI agents and scripts running corgi non-interactively.

## Non-interactive mode

Corgi auto-detects when it must not prompt and either skips the prompt or exits
with a clear error (exit code 2) instead of hanging. It is triggered by any of:

- A CI env var (`CI=true`, etc.).
- An agent env var: `CLAUDECODE`, `CLAUDE_CODE`, `ANTHROPIC_AGENT`.
- No TTY on stdin **or** stdout (piped / redirected).

Force prompts back on with the global `--interactive` flag.

## JSON output

Global `--json` makes stdout pure machine-readable JSON; human/log lines go to
stderr. Commands that emit pure-JSON stdout:

- `status --json` (one-shot: array; `--watch`: NDJSON, one object per transition)
- `doctor --json` (also `doctor --fix --json` → `{"ok":bool,"fixed":[...],"skipped":[{"check","reason"}]}`)
- `list --json`
- `config --json`, `config path --json`
- `ps --json`
- `open --json` → `{"opened":[{"service","url"}]}`
- `logs --json` → NDJSON, one `{"service","ts","level","line"}` per line (works with `--all` and follow)
- `db shell <svc> -e "<query>" --json` → `{"service","output"}`
- `create --kind ... --json` (non-interactive) → `{"created","kind","name","path"}`
- `restart --service <name> --json` → updated run-state object
- `mission-control --json` (one MissionSnapshot object; `--watch`: one object per refresh)
- `autopilot status/pause/resume/stop/heartbeat --json` → the autopilot loop state object (`{mode, iteration, lastHeartbeat, lastSummary}`); `mode` ∈ `uninitialized` (no state file yet — a first run, distinct from a stop) · `running` · `paused` · `stopped`
- `memory list --json` (array of facts), `memory lint --json` (`{"ok":bool,"errors":[...],"warnings":[...]}`), `memory add --json` / `memory index --json` (created/index summary)
- `suggest-history list --json` (`{"version","entries":[...]}`), `suggest-history check --slug <s> --json` (`{"skip":bool,"reason":"filed|dismissed|proposed|rate-limit|...","slug":...}`), `suggest-history record --json` (echoes the written entry), `suggest-history config --json` (`{"autoFileDrafts":bool,"maxPerWeek":n}`)
- `docs --json-schema`

Not yet pure-JSON: `fork` and the `db` bulk lifecycle flags (`--upAll` etc.)
still stream human output; use `corgi_up`/`corgi_down` (MCP) for lifecycle.

`run --json` is best-effort: it prints one JSON startup summary
(`{"started":[...],"failed":[...]}`) and then streams service logs to stderr —
stdout is not a single JSON document for the whole run.

Errors under `--json` have the shape:

```json
{"error": {"code": "E_INTERACTIVE_REQUIRED", "message": "..."}}
```

The `code` is a stable string an agent can branch on (see [Error codes](#error-codes)).

## Lifecycle (detached)

`corgi run --detach` (`-d`) starts every service as a detached process group that
survives corgi exiting, persists `corgi_services/.state.json`, and returns
immediately. It forces logs on. Under `--json` it prints the run-state object:

```json
{
  "composePath": "/path/corgi-compose.yml",
  "startedAt": "2026-05-21T10:59:47Z",
  "services": [
    {
      "name": "api",
      "kind": "service",
      "pid": 76960,
      "pgid": 76960,
      "command": "sleep 60",
      "logFile": "/path/corgi_services/.logs/api/2026-05-21T13-59-47.log",
      "status": "running",
      "startedAt": "2026-05-21T10:59:47Z",
      "statusChangedAt": "2026-05-21T10:59:47Z"
    }
  ],
  "dbServices": []
}
```

A second `run --detach` while a run-state exists errors (exit 1):

```json
{"error": {"code": "E_ALREADY_RUNNING", "message": "corgi is already running for this project — stop or restart first (use --force to override)"}}
```

`--force` replaces the existing run-state **and kills the previously tracked
processes first** (no orphans), then starts fresh.

### Status while detached

There is no daemon. With a state file present, `ps`/`status` report **real**
status (`running`/`crashed`/`stopped`) reconciled live — a dead pid flips to
`crashed` on the next read. Without a state file they fall back to declared
topology + a port probe. `statusChangedAt` lives in `.state.json` / the
`run --detach --json` run-state object, not in the `ps` rows (which carry just
`name`/`kind`/`port`/`status`/`url`).

```json
[{"name": "api", "kind": "service", "status": "running"}]
```

After the pid dies, the very next `corgi --json ps` shows:

```json
[{"name": "api", "kind": "service", "status": "crashed"}]
```

### Stop / restart

`corgi stop [--service <name>] [--json]` reads the state, SIGTERMs each process
group (SIGKILL after a grace period), runs `afterStart` hooks, brings
`db_service` containers fully down, and removes `.state.json`. `--service x`
stops one and keeps the rest. It is idempotent (exit 0 when nothing is running).

```json
{"stopped": ["api"], "failed": []}
```

`corgi restart [--json]` is a full-stack stop + detached start.
`corgi restart --service x` restarts a **single** detached service, leaving the
rest running. It only acts on a service already present in the detached
run-state — restarting one that was never started returns `E_NOT_RUNNING`:

```json
{"error": {"code": "E_NOT_RUNNING", "message": "service \"web\" is not in the current detached run; start it with corgi run --detach first"}}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Andriiklymiuk/corgi](https://github.com/Andriiklymiuk/corgi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
