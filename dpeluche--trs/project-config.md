---
trigger: always_on
description: A Rust CLI that transforms noisy terminal output into compact, structured signal.
---

# AGENTS.md — trs (Token-Reducing Shell)

## What is trs

A Rust CLI that transforms noisy terminal output into compact, structured signal.
Reduces token consumption by 68-99% for developers, AI agents, and automation pipelines.

## Pre-generated codebase digest

[`docs/development/codebase-digest.md`](./docs/development/codebase-digest.md)
is a snapshot of the entire trs codebase produced by `trs ingest`.
Drop it into any agent's context for an instant map of the project
without having to run `trs ingest` yourself.

The digest can drift from HEAD between releases. Regenerate before
tagging a release, or whenever `src/` has moved meaningfully:

```bash
./scripts/sync-codebase-digest.sh
```

The script uses `trs` from `PATH` and falls back to
`./target/release/trs` if one isn't installed.

## Related commands worth knowing

- `trs stats` — cumulative savings across commands (shows date range, today
  vs. average, last command, top-reducers).
- `trs doctor` — installation health check. Warns when AGENTS.md / CLAUDE.md
  exceed ~5k tokens and points at `trs audit-docs`.
- `trs audit-docs` — static analysis of agent instruction files
  (CLAUDE.md, AGENTS.md, rules files). Surfaces cross-file duplicates,
  embedded code/SQL/JSON blocks that belong elsewhere, dead `@imports`,
  and — for code fences — cross-references declared symbols against
  the actual source tree (so you can REMOVE ones already defined in src/
  and EXTRACT ones that don't live anywhere yet).
- `trs output-saver` — install a short output-reduction rules block
  into each agent's global config (AGENTS.md / CLAUDE.md / Cursor rules).
  Closes the symmetric gap: trs compresses what agents SEE via
  `trs rewrite`; output-saver compresses what they EMIT via
  anti-preamble / anti-narration / structured-output directives.
  Check-first by default (`--install` to commit, `--remove` to undo).
- `trs ingest` — project digest for LLM consumption. Use symbol index,
  compression levels, or `owner/repo` URL shorthand.
- `trs init --show` — see which AI agents have trs hooks installed.
- `trs upgrade` — detects the install channel (npm / curl) and re-runs
  it for the latest release. `--check` dry-runs, `-y` skips the
  confirmation prompt. See [`docs/features/upgrade.md`](./docs/features/upgrade.md).
- `trs init <tool>` — now runs a collision pre-check: detects hooks
  from other token-compression tools (via `@imports` too) and aborts
  by default. `--replace` removes competitor hooks cleanly, `--force`
  installs alongside (risky — double-compression can corrupt output).
  See [`docs/support/other-token-savers.md`](./docs/support/other-token-savers.md)
  for the list of tools we coexist with.
- `trs stats --by-agent` — breakdown by which AI agent fired each
  rewrite. Reads the `TRS_AGENT` env var that `trs rewrite` and
  plugin templates inject into the command line. Rules-only agents
  and direct shell runs show as `(untagged)`.
- **TRS_SKIP=1 prefix** — per-invocation bypass. Agents (or users)
  that need raw output for a specific command can prefix
  `TRS_SKIP=1 <cmd>`; `trs rewrite` passes it through unchanged.

## Architecture

```
src/
├── main.rs                    # Entry point, mod declarations
├── cli.rs                     # Cli struct, OutputFormat enum, flag precedence
├── commands.rs                # Commands enum, TestRunner
├── commands_parse.rs          # ParseCommands enum
├── command_registry.rs        # Single source of truth: per-command facts
│                              #   (aliases, rewrite/known, keep_ratio, stderr)
├── classifier.rs              # Subcommand → parser dispatch (reads registry)
├── classifier_exec.rs         # Execute → parse → format pipeline
├── classifier_transfer.rs     # Compact git push/pull/fetch output
├── config.rs                  # Config system (~/.trs/config.toml)
├── ingest/
│   ├── mod.rs                 # IngestConfig, DigestFile, run_ingest, resolve_project_root
│   ├── collect.rs             # File walker, read_and_compress, apply_budget
│   ├── deps.rs                # Import graph: extract_raw_imports, build_dep_graph, format_dep_*
│   ├── format.rs              # build_digest, build_tree, format_bytes/tokens
│   ├── ollama.rs              # Ollama post-processing (ollama_format)
│   └── store.rs               # ~/.trs/ingest/ persistence (save, list, read)
├── discover.rs                # trs discover — scan history for missed savings
├── init.rs                    # trs init — hook installer for 9 AI agents (see docs/development/agent-integrations.md)
├── rewrite.rs                 # trs rewrite — hook command rewriter engine
├── help.rs                    # Help text for all commands
├── process.rs                 # Process execution (spawn, capture, timeout)
├── process_helpers.rs         # Spawn error classification, output capture
├── tracker.rs                 # Token savings tracker (history.jsonl)
├── formatter/
│   ├── mod.rs                 # Formatter trait + select_formatter
│   ├── compact.rs             # Human-readable compact output
│   ├── compact_schema_git.rs  # Compact format: git status/diff schemas
│   ├── compact_schema_output.rs # Compact format: ls/grep/find/test/logs schemas
│   ├── json.rs                # Structured JSON

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dPeluChe/trs](https://github.com/dPeluChe/trs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
