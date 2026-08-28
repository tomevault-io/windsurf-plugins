---
trigger: always_on
description: `omp-prompt-monitor` is a local dashboard for [Oh My Pi](https://github.com/can1357/oh-my-pi) (omp): it scans every top-level session `.jsonl` file under `~/.omp/agent/sessions` (or a configured agent dir), extracts user prompts and text-bearing agent replies, and serves a live-updating HTML dashboard grouped by project/repo. It ships both as a standalone CLI (`omp-monitor`) and as an omp plugin (`/monitor` command) that can auto-launch the dashboard server from inside a session.
---

# Repository Guidelines

## Project Overview

`omp-prompt-monitor` is a local dashboard for [Oh My Pi](https://github.com/can1357/oh-my-pi) (omp): it scans every top-level session `.jsonl` file under `~/.omp/agent/sessions` (or a configured agent dir), extracts user prompts and text-bearing agent replies, and serves a live-updating HTML dashboard grouped by project/repo. It ships both as a standalone CLI (`omp-monitor`) and as an omp plugin (`/monitor` command) that can auto-launch the dashboard server from inside a session.

Security-sensitive: the server binds `127.0.0.1` only, but `/api/session/:id` and `--json` output expose **full prompt and reply text**. Never widen the bind address or add auth-bypassing routes without treating this as a security-relevant change.

## Architecture & Data Flow

The pipeline is one-way; on-disk and API contracts are versioned explicitly with `v: 1`. The index cache stores list metadata and user prompts only; full assistant replies are parsed on demand for one selected session.

```
journal.ts          index-store.ts        view.ts              server.ts / extension
parseSessionText  →  createIndexStore  →  buildSnapshot       →  /api/snapshot, /api/session/:id
(one .jsonl file)    .refresh()           (SessionParse         (strips prompts from list view,
 → SessionParse|null  (glob + mtime/size    + git + heartbeat     keeps them in detail view)
                       cache, bounded        → MonitorSnapshot)
                       concurrency)                              extension/index.ts writes a
                                                                  heartbeat file per session and
                                                                  registers /monitor
```

- **`src/core/journal.ts`** — `parseSessionText(text)` parses one session's JSONL: an optional v1 title-slot record, then a session header (must have a string `id` or the whole parse returns `null`). Walks records for user-authored `PromptRow`s (`kind: "typed" | "skill"`) and text-bearing assistant replies (`kind: "reply"`) in one chronological timeline. Synthetic/steering/agent-attributed/empty user entries and tool-only assistant entries are skipped; skill prompts additionally require `custom_message` + `customType === "skill-prompt"` + user attribution. Title precedence: slot title → header title → first user prompt truncated to 60 chars → `Session <id8>`. `deriveTailStatus` inspects the last message to classify `complete | interrupted | aborted | error | pending | unknown`. Malformed JSON lines are counted (`malformedLines`), not thrown.
- **`src/core/index-store.ts`** — `createIndexStore({ sessionsRoot?, cacheFile?, concurrency? })` returns an `IndexStore` whose `refresh()` globs `*/*.jsonl` (exactly one directory level — **no subagent-tree recursion by design**), reuses cached `IndexedSession` entries when `size`+`mtimeMs` are unchanged, parses with replies disabled, and persists only user prompts in `{ v: 1, entries }` to `<state dir>/index-cache.json` via `Bun.write` (skipped when nothing changed). Per-file stat/read/parse failures are swallowed and filtered out; unexpected glob/write errors propagate.
- **`src/core/view.ts`** — `buildSnapshot({ sessionsRoot? })` runs the lightweight index refresh and `readHeartbeats()` in parallel, enriches each session with git info (`describeRepo`) and `liveness` (`live` if heartbeat present, else `recent` within `RECENT_WINDOW_MS` = 15 min, else `idle`), sorts sessions by last activity, and groups them into `ProjectGroup`s keyed by `repoRoot ?? cwd`. `loadSessionPrompts(session)` re-parses one source file on demand for the full typed/skill/reply timeline and falls back to the indexed prompts if that file is unavailable.
- **`src/core/heartbeat.ts`** / **`src/core/paths.ts`** — heartbeat files live at `resolveHeartbeatDir()/<sessionId>.json`, written every 20s, considered stale after 60s; `readHeartbeats()` opportunistically unlinks stale or dead-same-host entries. `paths.ts` resolves all directories from env (`PI_CODING_AGENT_DIR`, `OMP_PROFILE`/`PI_PROFILE`, `PI_CONFIG_DIR`, `XDG_DATA_HOME`, `XDG_STATE_HOME`) with a strict precedence order — read this file before changing where anything is stored.
- **`src/core/git.ts`** — `describeRepo(cwd)` shells out to `git` via `Bun.spawn` with a 2s `AbortSignal.timeout`, memoized per-`cwd` for the process lifetime (including failed lookups); any git error/non-zero exit yields `null`, never a throw.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robwilde/omp-prompt-monitor](https://github.com/robwilde/omp-prompt-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
