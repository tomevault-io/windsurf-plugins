---
trigger: always_on
description: <!-- V_AGENTS.md  v1.1  2026-02-04 -->
---

<!-- V_AGENTS.md  v1.1  2026-02-04 -->

# V Repo Guide

Practical quick reference for the V compiler, standard library, and tools.
Written for AI coding agents; useful for humans too.

## Contents
* Quick Start
* Top Rules
* Agent Rules
* Safety (Do Not Brick the Repo)
* Divergences From Repo Docs
* Quick Decisions
* Common Workflow
* Reporting
* Prerequisites
* Build & Rebuild
* Run Programs
* Testing
* Code Style
* Modules and Imports
* C/JS Interop Hygiene
* Environment-Specific Code (files and `$if`)
* Compile-Time Code and Reflection
* Debug
* Compiler Architecture
* Key Directories
* Test Locations
* Error Reporting (checker/parser)
* Option/Result Types
* Tools
* Commits and PRs
* Environment Variables
* Gotchas

## Quick Start
Get operational from the repo root in three steps:

1. Build once (only if `./v` is missing): `make`
2. Build a working compiler:
   * Debug-friendly (recommended): `./v -g -keepc -o ./vnew cmd/v`
3. Use `./vnew` for everything:
   * Run a file: `./vnew run examples/hello_world.v`
   * Run tests: `./vnew -silent test vlib/v/`
   * Format: `./vnew fmt -w path/to/file.v`

Then read Top Rules and Agent Rules before making changes.

## Top Rules
* Use `./v` only to build `./vnew`; use `./vnew` for everything else.
* Put all V flags immediately after `./vnew` and before the
  subcommand/file, e.g. `./vnew -g run file.v`
  (not `./vnew run file.v -g`); flags after the subcommand are passed
  to that subcommand.
* Rebuild `./vnew` after compiler or core module changes
  (see Build & Rebuild).
* Run the smallest relevant tests; see Testing for triggers and
  minimums.
* Ask before large refactors or wide file touches (see Agent Rules).
* Do not stash or modify unrelated files unless explicitly instructed.
* This guide assumes agents run locally, not in CI; CI notes are
  informational only.
* When a summary is required, include behavior change, tests run, and
  touched file paths.
* If instructions overlap, prefer Build & Rebuild, Testing, and
  Reporting.
* If duplicates drift, treat Build & Rebuild, Testing, and Reporting as
  canonical and align other sections to them.

## Agent Rules

### Repo root
All commands assume the repo root as the working directory. The default
location is `/opt/v`, but this may differ in your environment. If a
command fails due to missing paths, verify with `pwd` and adjust
accordingly. Use a per-command workdir only when a task requires a
subdir.

### Tone and output
* Be concise by default. If the user asks for depth, provide it while
  keeping structure tight.
* Keep output easy to scan: short sections, bullets when listing,
  commands in backticks, no filler.
* Use a strict, operational tone unless higher-priority instructions
  override it.
* Ask only when required. If information is missing, ask a direct
  question.

### File access and edit scope
* You may read and edit all files in the V repo without asking for
  permission. This is file access, not change scope; scope is
  constrained below. Reading is always OK.
  Edits to `ci/` or `Dockerfile*` still require an explicit ask.
* Run build, test, and format commands without asking for permission.
  These are validation steps, not code changes. Only ask about edit
  scope, not about running `fmt`, targeted tests, or `check-md`.
* Only modify files required for the user request; avoid unrelated
  refactors. If duplication is harmful, small refactors to remove it
  are OK only when needed. Only refactor duplication in code you are
  already touching, and only when it directly supports the request or
  fixes a bug there. "Touching" means files already modified for the
  request.
* Avoid unrelated file changes; call them out if present.
* Avoid touching `thirdparty/` unless explicitly requested. If changes
  are needed there, ask for approval before proceeding.

### When to ask
* Ask before large refactors or wide file touches across multiple repo-root
  directories like `cmd/`, `vlib/`, `doc/`, and `examples/`.
  Exception: docs-only changes across many files are OK without asking;
  call them out in the summary.
* Ask before large behavioral changes within a single subsystem or file,
  even if the file count is small. Examples: changes to parser rules,
  checker resolution, codegen output shape, diagnostic text/ordering, or
  tool CLI behavior. Large means user-visible changes in CLI flags,
  output, diagnostics, or codegen shape. If unsure whether a change is
  "large," ask.
* Ask before touching `ci/` or `Dockerfile*` unless explicitly
  requested. If changes are needed there, confirm whether local
  validation is expected or if CI-only coverage is acceptable.

### Bootstrap
* Bootstrap/compiler usage rules: see Top Rules and Build & Rebuild.

### Completeness and summaries
* If you cannot complete a requested step, state the blocker and partial
  progress (what was attempted and what remains).
* After substantial work, provide a short summary and list touched file
  paths. Substantial work means any behavioral change, or changes in
  more than one file. Always include tests run (or "Not run" with
  reason) in the summary.
* Never change tests just to silence failures. Update expectations only
  when behavior changes are intended, and note the rationale in the
  summary.

### New files and deeper guidance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vlang/v](https://github.com/vlang/v) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
