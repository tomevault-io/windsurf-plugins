---
trigger: always_on
description: Guidelines for coding agents operating in this repository.
---

# AGENTS.md

Guidelines for coding agents operating in this repository.

## Instruction Source

`AGENTS.md` is the authoritative repository instruction file for this project.

`CLAUDE.md` is a thin bootstrap that defers to `AGENTS.md` and must not duplicate repository rules.

Keep product usage docs in `README.md`. Keep this file focused on how agents should work in the repository.

## Project

This repository contains `cc-statusline`, a configurable statusline command for Claude Code.

The current stack is:

- Node.js 18+ for the npm CLI and Windows runtime.
- Bash 3.2+ for Unix runtime and installer scripts.
- `jq` 1.6+ for JSON parsing and config merging.
- JSON presets under `presets/`.
- Shell smoke tests under `tests/`.
- macOS and Linux support through the Bash runtime and Homebrew.
- Windows support through the npm Node.js runtime.

Current repository shape:

- `statusline.sh`: Claude Code statusline entrypoint. Reads session JSON from stdin and prints one or more statusline rows.
- `lib/colors.sh`: ANSI color capability detection and color helpers.
- `lib/config.sh`: user config and preset loading.
- `lib/git.sh`: bounded git helpers used by statusline modules.
- `lib/modules.sh`: statusline module implementations.
- `lib/render.sh`: layout renderer that dispatches configured modules.
- `presets/*.json`: built-in layouts and default module settings.
- `install.sh`: installer that copies files, writes user config, updates Claude settings, and creates backups.
- `uninstall.sh`: uninstaller and backup restore flow.
- `tests/test.sh`: fixture-based smoke and regression tests.
- `tests/fixtures/*.json`: representative Claude Code session payloads.

## Language And Writing

All repository artifacts should be written in English unless the user explicitly asks otherwise.

Use ASCII punctuation by default. Do not use em dashes or en dashes in prose, comments, commit messages, plans, or documentation. Use the ASCII hyphen instead.

Unicode glyphs are allowed when they are part of the product output, screenshots, fixture expectations, or README examples for the statusline itself.

Use clear product terms: `statusline`, `module`, `preset`, `fixture`, `config`, `installer`, `uninstaller`, `settings`, `backup`, `workspace`, `context`, and `rate limit`.

## Documentation Lookup

Use Context7 MCP whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service. This includes Bash-adjacent tooling when current syntax or behavior matters, such as `jq`, `git`, shellcheck, packaging tools, or Claude Code configuration if available through documentation sources.

Workflow:

1. Resolve the library ID first unless the user provides an exact `/org/project` Context7 ID.
2. Query the selected documentation with the user's actual question.
3. Base the answer or implementation on the fetched documentation.

Do not use Context7 for ordinary refactors, project-specific shell debugging, code review, or business logic that can be answered from the local code.

## Core Design

Keep the runtime simple and predictable:

- `statusline.sh` reads exactly one JSON payload from stdin.
- Runtime code should render quickly and avoid network calls.
- Modules should return empty output when data is absent so separators do not dangle.
- Configuration should come from JSON presets plus user overrides.
- Presets should remain small, readable, and safe to merge with `jq`.
- The installer must be idempotent and must protect existing user configuration.

The statusline is a command-line rendering tool, not a TUI, daemon, service, or package manager. Do not add long-running processes or hidden background behavior.

## Shell Coding Rules

Prefer portable Bash that works with Bash 3.2 on macOS.

- Use `#!/usr/bin/env bash` for executable shell scripts.
- Keep `set -uo pipefail` unless a specific script needs a documented exception.
- Quote variable expansions unless word splitting is intentional.
- Prefer functions with narrow responsibilities over large inline blocks.
- Keep global variables namespaced with `CCSL_` when they are shared across files.
- Avoid Bash features unavailable in Bash 3.2.
- Avoid GNU-only command flags unless a macOS-compatible fallback exists.
- When formatting dates, keep both BSD/macOS `date -r` and GNU/Linux `date -d` behavior in mind.
- Do not use Python, Node, or another runtime for product code unless the project deliberately changes scope.

For JSON, use `jq` instead of ad hoc string parsing.

For git status, keep commands bounded and quiet. Git integration must not fail the whole statusline when the current directory is not a repository or git is unavailable.

## Module Rules

Modules live in `lib/modules.sh` as `mod_<name>` functions.

Module expectations:

- Read from `INPUT_JSON`.
- Read configuration through `cfg` or `cfg_raw`.
- Print only the rendered module text to stdout.
- Return success with empty output when the module should be hidden.
- Never print diagnostics to stdout.
- Handle missing, null, or partial Claude Code fields gracefully.
- Keep expensive or external operations out of modules unless they are bounded and optional.

When adding a module:

1. Add `mod_<name>` in `lib/modules.sh`.
2. Add default config only where needed in the relevant preset.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tenondecrpc/cc-statusline](https://github.com/tenondecrpc/cc-statusline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
