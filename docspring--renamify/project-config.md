---
trigger: always_on
description: The ONLY reason you should stop is if you have a blocking question for the user. Otherwise keep executing the workflow loop until done.
---

## CRITICAL: YOU ARE AN AUTONOMOUS AGENT. DO NOT STOP WORKING UNTIL ALL TASKMASTER TASKS ARE COMPLETE. DO NOT EVER "PAUSE" AFTER A TASK IS COMPLETED AND WAIT FOR THE USER TO START YOU AGAIN. START THE NEXT TASK IMMEDIATELY.

The ONLY reason you should stop is if you have a blocking question for the user. Otherwise keep executing the workflow loop until done.

---

# Renamify - AI Agent Guide

Smart case-aware search and replace across code and files with atomic apply and undo.

## CRITICAL: Context-Specific CLAUDE.md Files

This repository contains multiple CLAUDE.md files throughout the codebase, each providing context-specific instructions for different parts of the project:

- **Root `/CLAUDE.md`** (this file) - Overall project architecture, workflows, and standards
- **`/renamify-mcp/CLAUDE.md`** - MCP server specific rules (Ultracite coding standards)
- **`/renamify-vscode/CLAUDE.md`** - VS Code extension specific rules and patterns
- **Other subdirectories may have their own CLAUDE.md files**

**You MUST read the context-specific CLAUDE.md file whenever working in that part of the codebase.** These files contain critical rules and patterns specific to that component. For example, the MCP server has strict TypeScript/accessibility rules enforced by Ultracite that don't apply to Rust code.

**Note:** `/renamify-e2e-test/` is a temporary directory created during e2e tests (a clone of the entire repo for testing self-renaming). It's not part of the source tree and should be ignored.

## Project background

- Greenfield, unreleased. No backwards compatibility. Delete and tidy old code immediately.
- Goal: ship a production-ready v1.0.0
- Prioritize developer UX and ergonomics
- Key insight: developers need cross-case content replacement plus file and directory renames in one atomic operation with preview and history.

## Project Creator / Maintainer

The company who is sponsoring the development of this project and maintaining it is: DocSpring.
They should be mentioned in any package files, copyright notices, etc.

## Deliverables

- Rust core library and CLI (`renamify-core`, `renamify-cli`)
- VS Code extension (TypeScript) that shells out to the CLI
- MCP server (TypeScript) that wraps the CLI and exposes tools for Cursor or other MCP clients

## Core principles

- Safety first: plan, preview, atomic apply, rollback, history
- Case intelligence: generate all variants for a token
- One-pass scan: compile a single regex and scan once
- Git-aware safety. Use git for safe points and optional commits. Do not touch `.git` internals directly
- DX over legacy. No backward compatibility constraints

## Technologies

- Rust for core and CLI
- Search stack: ripgrep ecosystem crates and friends
  - `ignore` for .gitignore and fast dir walking
  - `globset` for include and exclude globs
  - `regex-automata` and `aho-corasick` for matching
  - `bstr` for fast byte string operations
- JSON for plan and history
- TypeScript for VS Code extension and MCP wrapper
  - `ts-rs` for automatic TypeScript type generation from Rust structs
  - Types are generated as ambient `.d.ts` files in `renamify-core/bindings/`
  - VS Code and MCP projects include these bindings directly in their tsconfig
- Optional Node bindings in future via `napi-rs` if needed

## Functional scope

- Case styles: snake_case, kebab-case, camelCase, PascalCase, SCREAMING_SNAKE_CASE, Title Case, Train-Case, dot.case
- Plan: generate all old variants, map to new variants, create a single search program, scan once, write `.renamify/plan.json`
- Apply: update file contents, then rename files and directories, all atomically
- Undo and redo: `.renamify/history.json` with checksums
- Conflicts: re-validate hunks, auto-resolve simple formatting shifts, stop on real conflicts unless forced
- Respect ignore files by default (`.gitignore`, `.ignore`, `.rgignore`, `.rnignore`), allow include and exclude globs
- Exclude binary files by default

## Non-goals for v1

- No AST or language-semantic renaming
- Only VS Code IDE integration
- Local execution only
- No telemetry by default

## Repo layout

- `renamify-core` - core logic
- `renamify-cli` - CLI frontend
- `renamify-mcp` - MCP server
- `renamify-vscode` - VS Code extension
- `docs` - Starlight documentation

## Agent roles and behavior

- Orchestrator: plan tasks, maintain dependency order, call Task Master commands
- Executor: implement code and tests, propose diffs, write files, update docs
- Checker: verify acceptance criteria, run linters and tests, measure coverage
- Always keep moving. After finishing a task, immediately pick the next task
- Ask the user only on blockers that cannot be resolved from repo context

## Definition of done

- Code compiles on macOS and Linux
- CI passes: format, clippy, tests
- 100 percent coverage for core operations
- Plan and apply work on sample repos with preview, atomic apply, and undo

## Quality bars

- Clippy warnings are errors
- `rustfmt` enforced
- Property tests for case conversions and boundaries
- Snapshot tests for plans and diffs
- Fuzz tests for regex generation to prevent backtracking issues
- Cross platform tests including Windows path edge cases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DocSpring/renamify](https://github.com/DocSpring/renamify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
