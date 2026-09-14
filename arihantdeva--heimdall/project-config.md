---
trigger: always_on
description: Heimdall is a **trust-verified, self-healing knowledge layer for AI coding agents**: it watches what an agent does, keeps a semantic-memory graph fresh across every project, and labels every search hit with a trust verdict (STRONG / WEAK / STALE) so an agent never acts on a dead path or a hallucinated match. It is the code behind the "kb_search before implementing" hard gate and the `~/knowledge-base`/Graft machinery the harness rules reference.
---

# AGENTS.md — Heimdall (working guide for AI agents)

## Purpose (one line)

Heimdall is a **trust-verified, self-healing knowledge layer for AI coding agents**: it watches what an agent does, keeps a semantic-memory graph fresh across every project, and labels every search hit with a trust verdict (STRONG / WEAK / STALE) so an agent never acts on a dead path or a hallucinated match. It is the code behind the "kb_search before implementing" hard gate and the `~/knowledge-base`/Graft machinery the harness rules reference.

The vendor repo lives at `/Users/arihantdeva/Repos/heimdall`; **live runtime state lives OUTSIDE the repo** under `~/.heimdall/` (journal, lock, hint queue, config) and `~/.graft/` (backend daemon + its sqlite DB), plus the `~/knowledge-base/` TSV/telemetry/stale logs. The repo is the engine; the home-dir files are the data.

## Structure map

| Path | Role |
|---|---|
| `bin/heimdall.js` | npm CLI entrypoint — thin dispatch to `bin/lib/cli-main.mjs` |
| `bin/lib/cli-main.mjs` | CLI dispatch + `init`/`insert`/`hint`/`verify`/`depth` implementations |
| `bin/heimdall-reconciler.mjs` | the **single writer** daemon: watch + hint ingest + drain + periodic audit |
| `bin/lib/journal.mjs` | authoritative index: sqlite (`node:sqlite`) — paths, owned nodes/edges, pending edges, dedup queue, generations |
| `bin/lib/reconcile.mjs` | level-triggered convergence: read disk, make graph match; `audit()` = drift detector |
| `bin/lib/extract.mjs` | desired state per file: hash + L0-L3 node/edge extraction, node-id namespacing, tree-sitter bridge |
| `bin/lib/heimdall_extract.py` | Python bridge: calls graphify per-language extractors directly (never `graphify.extract()`) |
| `bin/lib/depth.mjs` | depth ladder (path/file/symbol/graph), capability probe, config, root matching |
| `bin/lib/lock.mjs` | O_EXCL single-writer lock with stale-PID reclamation |
| `bin/lib/hints.mjs` | the only channel a non-writer may use: append "look at this path" lines |
| `bin/lib/sink.mjs` | projection targets: `MemorySink` (tests) and `GraftSink` (graft CLI) |
| `bin/lib/adapters.mjs` | `heimdall init --harness X` config writers (pi, claude-code, codex, cursor, windsurf) |
| `bin/kb-search.sh` | ranked search: graft retrieve + verify + graft explore walk |
| `bin/kb_search_verify.py` | trust verdicts: STRONG/WEAK/STALE/REBUILT/REMOVED/NOPATH, content-aware, path extraction, stale handling |
| `bin/kb-stale-scan.py` | full-graph stale sweep: rehome via `kb-rehome.sh` or log+delete |
| `bin/kb-rehome.sh` | deterministic rehome of a stale node (bounded basename search) |
| `bin/kb-health.sh` | `heimdall doctor`: daemon up, CLI responsive, search smoke, inventory freshness |
| `bin/kb-rebuild.sh` | full graph rebuild: backup → wipe → parallel restore → re-seed → prune → verify |
| `bin/seed-graft.sh` | load `~/knowledge-base/.inventory.tsv` into Graft (idempotent) |
| `bin/sync-edits.sh` | one-shot bootstrap: replay Pi session edit logs (write/edit/hashline_edit) as hints |
| `bin/telemetry.sh` | `collect`/`view`/`usage` — nodes/day, sync age, kb_* tool-call counts |
| `extensions/kb-tools.ts` | Pi extension: exposes `kb_search`/`kb_insert`/`kb_sync` agent tools |
| `extensions/kb-autosync.ts` | Pi extension: hook that appends path hints (never writes the graph) |
| `extensions/kb-orient.ts` | Pi extension: injects prior-work hits into the first user prompt of a session |
| `extensions/kb-search-guard.ts` | Pi extension: warns/escalates/blocks only on UNscoped discovery chains; agent-callable `kb_guard_pause` suspends enforcement 1–20 turns |
| `extensions/lib/kb-guard-core.mjs` | pure guard state machine (testable without the Pi runtime) |
| `vendor/graphify/` | vendored graphify v0.3.17 (MIT) — per-repo code-graph extractors (tree-sitter) |
| `vendor/graft/` | vendored Graft source (Apache 2.0) — backend daemon; **not prebuilt**, build from source |
| `config/heimdall.yaml.example` | example backend config → copy to `~/.graft/config.yaml` |
| `launchd/com.heimdall.backend.plist.example` | launchd template for `graftd` |
| `docs/adapters.md` | what each `heimdall init --harness X` installs |
| `docs/heimdall_compare.{dot,png}` | graphify vs Graft vs Heimdall positioning diagram |
| `types/pi-coding-agent.d.ts` | minimal type stub of the Pi host API so extensions typecheck standalone |
| `tests/*.test.mjs` | node:test suites (44 tests) |
| `.pi-subagents/` | subagent run artifacts (input/output/transcripts/meta) — gitignored, historical record |

## Entry points

- **CLI:** `node bin/heimdall.js <command>` or the installed `heimdall` binary. Subcommands: `init`, `search`, `insert`, `doctor`, `daemon`, `reconcile`, `verify`, `depth`, `hint`.
- **Read the flow here:** `bin/heimdall.js` → `bin/lib/cli-main.mjs` (dispatch) → `bin/heimdall-reconciler.mjs` (daemon loop) → `journal` / `hints` / `reconcile` / `extract` / `depth` / `sink`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArihantDeva/heimdall](https://github.com/ArihantDeva/heimdall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
