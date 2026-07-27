---
trigger: always_on
description: `openapi-changes` is a pb33f tool built by Princess Beef Heavy Industries, LLC.
---

# AGENTS.md

`openapi-changes` is a pb33f tool built by Princess Beef Heavy Industries, LLC.

This file is the fast path for agents and maintainers. Read this before changing command behavior, report semantics, or UI payloads.

Public documentation:

- https://pb33f.io/openapi-changes/

## What This Repo Is

`openapi-changes` is a Go CLI for comparing OpenAPI specifications across:

- direct left/right file, URL, or git-revision comparison
- local git history for a file in a repository
- GitHub-hosted file history via file URL

The public comparison/report surface is:

- `console`
- `summary`
- `report`
- `markdown-report`
- `html-report`

These five commands use the current engine built on `doctor`, the open source Apache 2.0 library from pb33f:

- https://github.com/pb33f/doctor

Utility commands:

- `completion`
- `version`

## Source Of Truth

The code is the source of truth.

If behavior, comments, tests, and internal docs disagree, bring them back into alignment in the same change. Do not leave drift behind.

## Architecture At A Glance

- `cmd/` owns CLI orchestration, shared flag handling, commit loading, doctor execution, JSON flattening, and report file writing.
- `git/` owns local git and GitHub history extraction and comparable commit/document preparation.
- `html-report/` owns the HTML payload contract, templates, and embedded frontend assets.
- `tui/v2/` owns the canonical Bubbletea console UI.
- `internal/changecounts/` owns deduplicated semantic count derivation.
- `internal/security/` owns security-scope labeling helpers.
- `github.com/pb33f/doctor` is the authoritative backend for changerator traversal, semantic tree rendering, markdown/HTML rendering, and GitHub history service integration.

All `cmd/` implementation files use their canonical names (e.g., `cmd/summary.go`, `cmd/console.go`).

## Non-Negotiable Invariants

### Counts and semantics

- `summary` is the semantic human report.
- `summary` headline, highlights, and visible summary counts are deduplicated by `JSONPath + property`.
- `markdown-report` top summary and object stats are also deduplicated.
- `report` is the machine-readable JSON interface and should prefer stable semantic identity over renderer-local structure.

### Machine-readable JSON

- `report` should emit semantic `path` values whenever possible.
- If semantic normalization rewrites an engine path, preserve the original path in `rawPath`.
- Parameter identity must not depend on array index positions when a stable semantic name exists.
- Historical `report` output must surface partial-success state in `metaData` instead of relying on stderr warnings alone.
- `metaData.skippedCommits` is the machine-readable list of git/GitHub history revisions that were skipped because they failed to normalize or render.

### Left/right behavior

- Left/right comparisons are synthetic comparisons, not fake git history.
- Do not emit synthetic commit metadata in left/right machine- or human-facing report output.
- Git revision inputs (`revision:path`) resolve `$ref` siblings from the same revision via `GitRevisionFS`, not from the working tree.

### Failure semantics

- Mixed-success histories should limp on with warnings when at least one comparable commit renders successfully.
- `summary`, `markdown-report`, and `html-report` should fail only when every candidate commit fails to render/build.
- Historical `report` output should limp on when at least one history item renders, and must set `metaData.partial = true` when any commits were skipped.
- “No prior comparable version” and “no changes found” are distinct states.

### HTML payload integrity

- The HTML payload must preserve the raw changed-node tree for the document/explorer views.
- Do not run summary-style pruning/transforms on the HTML tree payload if they can drop valid branches such as `$.components`.

## Change Strategy

- Keep commands thin.
- Put shared logic in shared helpers instead of copying per-command behavior.
- Prefer using doctor as the rendering source of truth instead of recreating semantic renderers locally.
- Prefer semantic output over diagnostic/internal output for human-facing commands.
- For machine-facing output, stable identity beats pretty formatting.
- If the counted thing and the displayed thing diverge, fix that before shipping.

## Files That Matter Most

- `cmd/root.go` — root Cobra command, CLI entry point, subcommand registration
- `cmd/version.go` — prints the raw build version string
- `cmd/common.go` — shared option flags and Lip Gloss styling for all doctor-based commands
- `cmd/loaders.go` — loads specs from files, URLs, and git history; progress tracking and error aggregation
- `cmd/engine.go` — wraps doctor changerator for API comparison; manages document resource cleanup and mutex-guarded breaking config
- `cmd/summary.go` — styled terminal summary with deduplicated breaking/addition/modification/removal counts
- `cmd/report.go` — machine-readable JSON report pipeline
- `cmd/markdown_report.go` — markdown-formatted change reports via doctor renderer
- `cmd/html_report.go` — self-contained interactive HTML reports with embedded assets
- `cmd/console.go` — launches the interactive Bubbletea terminal UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pb33f/openapi-changes](https://github.com/pb33f/openapi-changes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
