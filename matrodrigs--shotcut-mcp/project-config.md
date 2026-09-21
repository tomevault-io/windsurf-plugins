---
trigger: always_on
description: validates it with the installed MLT runtime, renders previews and final media, and
---

# AGENTS.md

This file defines the engineering practices for automated and human contributors to
Shotcut MCP. Apply these rules to the entire repository unless a more specific
`AGENTS.md` exists below the file being changed.

## Project intent

Shotcut MCP is a dependency-free Python MCP server that edits Shotcut MLT XML,
validates it with the installed MLT runtime, renders previews and final media, and
preserves user projects through transactional writes. Correctness and preservation of
user data take priority over convenience or clever abstractions.

New projects target Shotcut 26.8.1 with MLT 7.41.0 serialization (7.41.x family).
`shotcut_mcp/__init__.py` separately defines exact tested runtime pairs; the Windows CI
matrix is generated from that list. To recognize a new pair, pin its archive checksum in
`scripts/compatibility_matrix.py` and pass real integration in the same change. Unknown
versions receive a warning rather than a version-based block. Keep concrete runtime
failures distinct, and check RNNoise independently as both a link and a filter.

## Architecture and dependency direction

Keep modules deep: expose a small interface that hides substantial behavior. Add a seam
only when behavior genuinely varies or when it isolates an external dependency. Avoid
pass-through wrappers, speculative ports, and circular imports.

Before changing module boundaries or imports, read the dependency graph and ownership table
in [docs/architecture.md](docs/architecture.md). Keep that document as the architecture reference.

Preserve the public imports from `project.py` and `platform.py`. Internal modules may be
reorganized without changing MCP clients, tool names, schemas, result shapes, or the
documented public imports exposed by those two modules.

## Non-negotiable safety invariants

- Treat every `.mlt`/`.xml` project and rendered output as user data.
- Never replace a project until the complete candidate passes MLT validation.
- Recheck the project revision after validation and before replacement.
- Use sibling temporary files, `fsync` where supported, and atomic replacement.
- Preserve an existing target if validation, preview, render, or promotion fails.
- Keep project backups isolated by canonical project identity. Restore only filenames
  produced by the backup store; directory membership alone is not ownership proof.
- Preserve unknown MLT XML elements and properties unless an operation explicitly owns
  them. Reject duplicate IDs and ambiguous timeline roots instead of guessing.
- Clone a shared producer before applying a clip-local mutation.
- Treat Shotcut marker `end` values as exclusive. Translate a range marker to MLT's
  inclusive consumer `out` by subtracting one, and reject empty ranges.
- Reject media replacement next to a transition instead of attempting to rewire its
  nested tractor. Preserve the entry range and isolate a shared producer before mutation.
- Apply the configured allowed-root and network-resource policy to all user-controlled
  data paths. Executable discovery is not a data-path operation.
- Do not invoke a shell for Shotcut, MLT, FFmpeg, or FFprobe commands. Pass argument lists
  directly and terminate the process group on timeout or MCP cancellation.
- Restrict render consumer properties to single-file-safe options unless the
  administrator explicitly enables unsafe properties.
- Keep MCP stdout strictly newline-delimited UTF-8 JSON-RPC. Diagnostics go to stderr.
- Emit request progress only for a caller-supplied token, keep values strictly increasing,
  and omit the progress `message` for MCP `2024-11-05`. A render token ends when
  `start_render` returns; durable progress remains owned by `render_status`.
- Bound message size, in-flight work, logs, diffs, operation batches, and caches.

## Change practices

- Make the smallest cohesive change that solves the observed problem.
- Diagnose bugs as symptom -> source -> consequence -> remedy before editing.
- Prefer extraction along an existing responsibility seam over splitting by file size.
- Keep edit rules in `project_document.py` and read projections in
  `project_snapshot.py`; keep transaction and filesystem policy out of both.
- Keep OS/process mechanics in `processes.py`; callers should not duplicate subprocess
  construction, cancellation polling, or executable discovery.
- Keep FFmpeg analyzer commands and stderr parsers in `media.py`. Report a missing filter
  or inapplicable stream per analyzer so one unavailable check does not erase useful results.
- Keep path authorization in `path_policy.py`; do not add ad hoc prefix checks.
- Keep MCP handlers thin and validate through the declared tool schema before execution.
- Route progress through `protocol.py` and serialize notifications in `server.py`; domain
  modules report semantic milestones without knowing tokens or protocol revisions.
- Preserve Python 3.10 compatibility and standard-library-only runtime code.
- Use type annotations for new interfaces and return structured dictionaries consistent
  with existing tool results.
- Do not silently weaken validation, revision checks, backup creation, path policy, or
  output protection to make a test pass.

## Agent-facing MCP guidance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matrodrigs/shotcut-mcp](https://github.com/matrodrigs/shotcut-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
