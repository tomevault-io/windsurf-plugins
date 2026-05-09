---
trigger: always_on
description: PocketBun is a **Bun-native** reimplementation/port of **PocketBase** with the goal of being **API-compatible** (drop-in for clients + Admin UI).
---

# AGENTS.md — PocketBun

## What this repo is
PocketBun is a **Bun-native** reimplementation/port of **PocketBase** with the goal of being **API-compatible** (drop-in for clients + Admin UI).
- Runtime target: **Bun only** (no Node.js support needed).
- Language: **TypeScript** (keep types practical; avoid type wizardry).
- Serve the existing PocketBase Admin UI **unchanged** as static assets.

PocketBun is a separate open-source project and must **clearly credit PocketBase** and preserve upstream license notices for any copied code/assets.

## Repository Guidelines
- Repo: https://github.com/pekeler/pocketbun
- Trunk-based development: work directly on `master` unless the repo owner explicitly asks for a branch. Do not create `codex/*` or other task branches by default.
- GitHub issues/comments/PR comments: use literal multiline strings or `-F - <<'EOF'` (or $'...') for real newlines; never embed "\\n".

## Top priority: compatibility
When implementing behavior, prioritize **matching PocketBase observable behavior**:
- routes, status codes, JSON response shapes
- query params semantics (filters, sort, expand, fields, paging)
- auth/token behavior
- realtime protocol behavior (SSE)
- error formatting
- JS developer-facing APIs (method names/casing and usage patterns) as documented by PocketBase JSVM

If you’re unsure about an edge case:
1) check PocketBase docs
2) check PocketBase source in the pinned upstream checkout (see below)
3) if still unclear, run upstream PocketBase at the pinned tag and compare behavior, then encode it in tests.
If you discover a behavior difference that cannot be avoided, confirm with the repo owner that the incompatibility is acceptable before documenting it. Only then add it to the README’s “Known Differences” list (keep that list very short).

## Porting Guidelines
Goal: maximize long-term maintainability and upstream-syncability by keeping PocketBun structurally close to the PocketBase Go codebase, while preserving *observable behavior*.

- **Behavior first:** Match PocketBase observable behavior (API, status codes, JSON shapes, error formats, auth, realtime) even if the internal implementation differs.
- **Mechanical translation preferred:** Avoid “cleanup”, refactors, or re-architecture unless needed for correctness or Bun constraints.
- **Traceability:** When porting a file, add a short header comment linking to the upstream source path (no version/hash; `pocketbase_tag.txt` is the source of truth), e.g.
  `// Ported from pocketbase/<path>`
  - If a single TS file merges multiple upstream files, list *all* upstream source paths in that header comment.
- **Preserve upstream comments:** Copy upstream comments (doc comments and relevant inline notes) into the TypeScript port. If comments are missing, backfill them so the TS file reflects upstream commentary.
- **Comment backfill is mandatory:** When you notice missing upstream comments in already ported code, add them immediately rather than deferring.
- **Non-upstream files:** Any source or test file without an upstream counterpart must include a short header comment explaining why the file exists (keep these files to a minimum).
- **1:1 file mapping (when reasonable):** Prefer one `.ts` file per corresponding `.go` file and mirror directory structure (Go packages → TS folders) to keep diffs and future syncing straightforward.
  - If strict 1:1 creates unnatural modules (circular imports, huge files, etc.), it’s OK to merge/split — but document it in a comment near the top of the file and list every upstream source file included.
- **Naming:** Prefer upstream naming and concepts for internal identifiers (types/functions), even if not idiomatic JS/TS, **as long as it doesn’t reduce clarity or cause bugs**.
  - For public JS/TS APIs, prioritize PocketBase JSVM naming/casing over Go exported naming.
  - Go-style names may exist as compatibility aliases, but must not be the only public API form when a JSVM equivalent exists.
  - Docs/examples should use JSVM-style names and casing.
  - Do not rename just for style.
- **Only deviate when necessary:** Deviations are allowed when required by JS/Bun semantics (async I/O, concurrency model, time/number handling, resource cleanup, etc.).
  - When you deviate, leave a brief comment explaining *why*.
- **Document significant differences:** If a ported file intentionally diverges from upstream behavior/implementation, document the difference and the rationale in the file.
- **Compatibility shims:** Prefer small internal helpers (`src/internal/compat/*`) to model Go-like primitives (errors/time/sync/http/sql) so most files remain a straightforward, 1:1 port.
- **Dependencies:** Prefer **Bun built-ins** first (e.g. `Bun.serve`, `bun:sqlite`, Web APIs).
  - If PocketBase uses a Go third-party library and Bun lacks equivalent functionality, choose an npm dependency that is:
    - actively maintained,
    - reasonably popular,
    - small/specific (avoid heavy frameworks),
    - permissively licensed (MIT/Apache/BSD).
  - Avoid adding dependencies for trivial utilities—write small local helpers instead.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pekeler/pocketbun](https://github.com/pekeler/pocketbun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
