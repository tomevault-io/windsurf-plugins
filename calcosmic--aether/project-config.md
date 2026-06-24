---
trigger: always_on
description: > **Current Version:** v1.0.34
---

# AGENTS.md -- Aether Development Guide (Codex CLI)

> **Current Version:** v1.0.34
> **Last Updated:** 2026-05-06
> **Platform:** Codex CLI (OpenAI)

This file provides project-level instructions for Codex CLI, equivalent to
`CLAUDE.md` for Claude Code. Aether supports three platforms: Claude Code,
OpenCode, and Codex CLI.

## Platform Policy

- **Primary platforms:** Claude Code and OpenCode. These are the main maintained user surfaces.
- **Secondary platform:** Codex CLI. Codex has best-effort support for the direct `aether` workflow.
- **What that means for Codex:** keep the native CLI lifecycle, install/update flow, state integrity, and worker dispatch safe and usable. Codex UX polish and wrapper semantics may lag behind Claude/OpenCode.

---

## Quick Reference

| What | Count/Status |
|------|--------------|
| Version | v1.0.34 |
| Agent definitions | 25 (TOML in `.codex/agents/`) |
| Skills | 86 (55 colony + 31 domain) |
| Go binary | `aether` CLI (Go binary in cmd/) |
| Verification | `go test ./...` and `go test ./... -race` clean |
| Architecture doc | `RUNTIME UPDATE ARCHITECTURE.md` |

---

## How Aether Works in Codex CLI

Codex CLI does not support slash commands. All colony operations use the `aether`
CLI binary directly or natural language prompts. There is no equivalent of
typing `/ant-build 1` -- instead you run:

```bash
aether build 1
aether plan
aether init "Build feature X"
aether continue
aether status
```

When the user types a literal `aether ...` command in Codex, execute that exact
CLI command first for literal passthrough commands such as `status`, `update`,
`focus`, `pheromones`, and `reference-list`.

Exception: `aether init`, `aether oracle`, `aether plan`, `aether build`,
`aether continue`, `aether seal`, and `aether discuss` have wrapper-equivalent
Codex orchestration. For those commands, run or inspect
`aether command-guide <command> --platform codex` and use the matching Codex
skill (`aether-colony-creation`, `aether-colony-research`, or
`aether-colony-build-cycle`) unless the user explicitly says raw, exact,
no-interview, or no-orchestration.

Use `aether --help` as the runtime source of truth if markdown docs disagree.
For lifecycle commands run through Codex shell execution, prefer
`AETHER_OUTPUT_MODE=visual aether ...` unless the user explicitly wants JSON.
Do not preface literal passthrough commands with repo archaeology, skill
narration, or "I'm checking..." commentary. The CLI output is primary; your own
wrapper should be zero or one short sentence.

Agent definitions live in `.codex/agents/*.toml` (TOML format) and Codex reads
them as part of its agent discovery system.

## UX Architecture

Codex gets its colony identity entirely from the Go runtime visual renderer
(`cmd/codex_visuals.go`). Unlike Claude/OpenCode, Codex does NOT use wrapper
markdown — the CLI output IS the presentation.

### Caste Identity System

Each worker caste has:
- **Emoji prefix** — decorative icon (e.g., `🔨` for Builder, `👁️` for Watcher)
- **ANSI-colored label** — the primary identity, colored by caste (e.g., "Builder" in yellow, "Watcher" in cyan)
- **Deterministic name** — hash-based per caste+task (e.g., "Mason-67", "Keen-6")

Visual format: `🔨 Builder Mason-67  Task description`

Color detection uses `NO_COLOR`, `AETHER_FORCE_COLOR`, `CLICOLOR_FORCE`, and TTY
checks. In non-TTY or JSON mode, colors are disabled and only plain text appears.

### Stage Markers

Build and continue output uses stage transition markers:
```
── Context ──
── Tasks ──
── Dispatch ──
── Verification ──
── Housekeeping ──
── Next Phase ──
── Colony Complete ──
```

### Output Modes

| Mode | When | Content |
|------|------|---------|
| Visual | `AETHER_OUTPUT_MODE=visual` or TTY | ANSI banners, emojis, colors, stage markers, context-clear guidance |
| JSON | `AETHER_OUTPUT_MODE=json` or piped | Structured data envelopes for programmatic use |

### Codex Orchestration Layer

Claude/OpenCode wrappers add colony framing, scoping interviews, worker
orchestration, and post-command narration on top of runtime output. Codex has no
slash-command wrapper files, so the equivalent layer is:

- `aether command-guide <command> --platform codex` for command-specific steps
- `aether-colony-creation` for init synthesis
- `aether-colony-research` for oracle/discuss synthesis
- `aether-colony-build-cycle` for plan/build/continue/seal orchestration

For pure visual/runtime polish, change `cmd/codex_visuals.go`. For intelligent
command behavior, keep the YAML source, Claude/OpenCode wrappers, Codex skill,
and `cmd/command_guide.go` aligned in the same change.

---

## Architecture Overview

```
+------------------------------------------------------------------+
|                     AETHER REPO (this repo)                       |
|                                                                   |
|   cmd/                 <- Go source code (primary)               |
|   +-- main.go         CLI entry point                            |
|   +-- *.go            Command implementations (80+ subcommands)  |
|                                                                   |
|   pkg/                 <- Shared Go packages                     |
|   +-- agent/          Agent pool, spawn tree, curation ants      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [calcosmic/Aether](https://github.com/calcosmic/Aether) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
