---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`openmemory-cli` (CLI command: `openmemory`) — a hub-and-spoke CLI + TUI that ports **conversations** between AI coding harnesses (Claude Code, Codex, OpenCode). v1 fidelity is **text turns only**; tool calls, tool results, and thinking blocks are dropped at import and counted, not carried.

All code lives under `cli/`. Runtime is **Bun** (not Node) — everything runs `.ts`/`.tsx` directly, no build step.

**Distribution is source-checkout only**: `scripts/install.sh` (curl-able) clones this repo, runs `bun install`, and drops a shim on PATH. There are deliberately **no binary releases, no npm publish, and no Homebrew formula** for now — don't add release/publish machinery without asking. Consequence: anything committed under `cli/src/` is public source (e.g. a PostHog project API key is fine — write-only — but real secrets never).

Put any generated markdown, plans, or TODO files under `claude-generated-docs/`.
Put any blog/article generated files in `blog-posts/`.

## Commands

Run from `cli/`:

```bash
bun install
bun test                                   # full suite (~90s; includes real round-trip e2e)
bun test test/detect.test.ts               # single file
bun test -t "some test name"               # by name filter
bun run typecheck                          # tsc --noEmit
bun run cli <command>                      # = bun run src/cli.ts
```

CLI subcommands (`src/cli.ts`): `detect`, `port --from <h> --to <h> (--all | --id <sid>...) [--force] [--json]`, `verify [--from] [--to a,b] [--sample N | --all]`, `tui`. There is deliberately **no** standalone `import`/`export` subcommand — in-memory Hub IR can't be passed between CLI invocations (dropped in S8).

### Sandbox commands (test harness)

Run from `cli/`. All target `~/.openmemory-sandbox/` (`OPENMEMORY_SANDBOX_ROOT` overrides) and never touch real stores unless a backup ran first. `--harness claude-code|codex|opencode|all` (default `all`).

- `bun run sandbox:backup --harness all` — snapshot real stores to timestamped backups.
- `bun run sandbox:restore --harness all [--from <timestamp>]` — restore latest/named backup.
- `eval "$(bun run sandbox:reroute --harness codex)"` — point the porter CLI at the sandbox (write mode; includes `OPENMEMORY_LEDGER_PATH` isolation).
- `eval "$(bun run sandbox:reroute --harness codex --app)"` — env for launching the desktop app against the sandbox (best-guess pending spike).
- `bun run sandbox:seed --projects 3 --convos 5 [--source-root <dir>]` — seed sandbox claude store from real convos.
- `bun run sandbox:clear --harness all` — wipe sandboxed conversations (sandbox paths only, guarded).

Verification helpers (modules, not CLIs): `scripts/sandbox/store-check.ts` (`sessionExists` via destination `discover()`), `scripts/sandbox/resume-probe.ts` (non-interactive resume: `claude --resume <id> -p`, `codex exec resume <id>`, `opencode run -s <id>`), `scripts/sandbox/issues.ts` (`logIssue`/`readIssues` → `issues.jsonl`). The full e2e flow is orchestrated by the `port-sandbox-test` skill — pure reporter, failures logged, fixes dispatched after the run.

## Architecture

The whole tool is a **hub-and-spoke** around one intermediate representation. Every import produces a `PortableSession`; every export consumes one. Harnesses never talk to each other directly.

- `src/ir.ts` — **the hub.** `PortableSession` Zod schema. `sourceSessionId` is the idempotency key; `cwd` is authoritative (never re-derive it from encoded path names); `sourceMetadata` is a lossless bag; `droppedTurns` counts tool/thinking-only turns skipped at import.
- `src/adapter.ts` — the contract each spoke implements. `ImportAdapter` = harness → IR (`detect`/`discover`/`parse`). `ExportAdapter` = IR → harness (`detect`/`write`/`teardown`).
- `src/adapters/{claude-code,codex,opencode}/` — one import + one export module per harness. `src/adapters/index.ts`'s `registerAllAdapters()` is the **only** place adapters touch the registry; adapters never register themselves.
- `src/registry.ts` — register/lookup adapters by harness + direction. `listInstalled()` intersects S7 detection with what's actually registered.
- `src/detect.ts` (S7) — pure, read-only, never-throws probing of which harnesses are present and read/writable. Takes an injectable `DetectEnv` so tests can stub `HOME`/`CODEX_HOME`/`PATH`/`XDG_DATA_HOME`.
- `src/core/orchestrator.ts` (S8) — the engine: detect → import → export pipeline over the registry, emitting a `PorterEvent` stream. **Zero TUI dependency by construction.** Per-session failures are isolated (emit `error`, never abort the batch).
- `src/core/ledger.ts` (S8) — **the single idempotency authority for the whole tool.** Keyed on the triple `(sourceHarness, sourceSessionId, destHarness)`. JSON file under `$XDG_DATA_HOME/openmemory/`. Writes are serialized per path via an in-process lock.
- `src/tui/` — SolidJS + `@opentui` terminal UI (not Ink). `app.tsx` is the 5-step wizard shell (Destination → Source → Select → Import → Done); `wizard.ts` holds all render-free, unit-testable logic; `screens/` are thin views over it.

## Decision wiki (`docs/brain/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mem0ai/openmemory](https://github.com/mem0ai/openmemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
