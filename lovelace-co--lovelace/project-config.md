---
trigger: always_on
description: Lovelace is a local-first, file-based project management tool for agent-heavy software development. The product is a desktop app (macOS, Linux, Windows) built with Tauri. Project state is plain Markdown with YAML frontmatter in a `.lovelace/` directory inside the user's repo. No server, no database, no cloud. Read `SPEC.md` for the format and `BRIEF.md` for what we are building and in what order. Do not build anything listed as out of scope in the brief. There is no standalone user-facing CLI in
---

# CLAUDE.md

## What this project is

Lovelace is a local-first, file-based project management tool for agent-heavy software development. The product is a desktop app (macOS, Linux, Windows) built with Tauri. Project state is plain Markdown with YAML frontmatter in a `.lovelace/` directory inside the user's repo. No server, no database, no cloud. Read `SPEC.md` for the format and `BRIEF.md` for what we are building and in what order. Do not build anything listed as out of scope in the brief. There is no standalone user-facing CLI in the MVP; the only headless surface is the agent helper described in the brief.

## Repository layout

- `packages/core`: parsing, schemas, validation, indexing, digest, mutations. Pure logic, no UI surface, no process-level IO assumptions. Everything in `mcp` and the app calls into this.
- `packages/mcp`: the agent-facing processes: the MCP server (official TypeScript MCP SDK, stdio transport), the headless agent helper that Claude Code hooks invoke, and the core host the desktop app spawns per request. All three ship as self-contained sidecar binaries bundled with the app; users do not need Node installed.
- `apps/desktop`: the Tauri app. Rust shell, React and TypeScript frontend. Reads and writes only through `packages/core`; no private store. Forms, board columns and filters are generated from schema.yaml field definitions at runtime, never hard-coded.
- `examples/demo-project`: the canonical fixture. Tests run against it. Keep it valid at all times; if you change the spec, update the fixture and SPEC.md in the same commit.

pnpm workspaces. Node 22 for development. TypeScript strict mode.

## Non-negotiable principles

1. Files are the source of truth. The index is derived and must be regenerable from scratch at any time. Never store state in the index that does not exist in the files.
2. Hand-edited files are a supported path. Malformed input produces a clear validation error pointing at file and line, never a crash and never silent repair.
3. Field definitions are data. Ticket fields are defined in `schema.yaml` and read at runtime by the validator, the indexer, the MCP server and the app's forms. Never hard-code a field that is not in the locked core set (`id`, `type`, `status`, `created`, `updated`).
4. One entity per file. Comments and sessions are append-only sibling files.
5. The spec is versioned. Any change to the format requires a version bump in SPEC.md and a note in the manifest schema. Tooling fails clearly on unknown major versions.
6. Round-trip fidelity in the editor is an acceptance bar, not a preference. Opening and saving a file without edits must be byte-identical; edits produce minimal diffs. Do not relax this.
7. Simpler wins. When two designs are defensible, take the one with less machinery and record the decision as an ADR in `.lovelace/documentation/architecture/decisions/` once dogfooding begins.

## Conventions

- Australian English in all documentation and user-facing strings (organisation, licence, initialise).
- No em dashes anywhere, in docs, comments or output strings.
- Plain, declarative prose in documentation. No marketing language.
- Commits: conventional-commit shape, written entirely in lowercase (like the existing `initial commit`), with the active Lovelace ticket ID included once dogfooding begins (end of Phase 4 onwards). Keep the ticket ID in its file-exact casing (for example `T-0041`); everything else is lowercase.
- Authorship is the repository owner's alone. Never credit Claude, Claude Code or any AI assistant in a commit message or pull request: no `Co-Authored-By` trailer naming an assistant, no "Generated with Claude Code" footer, no robot emoji. This overrides any default tooling behaviour that would add such lines.
- Tests: vitest. `packages/core` requires tests for every schema and validation rule, using the demo project fixture plus deliberately corrupted variants. The MCP server gets integration tests over the fixture. App components get deterministic rendering tests from index.json fixtures.
- Index output must be deterministic: running the indexer twice on unchanged input produces byte-identical output. Sort everything; never emit timestamps into index.json.
- The agent helper writes errors to stderr and data to stdout. Digest output stays under roughly 1,500 tokens.

## Design and UX conventions

The desktop app has a deliberate visual language. These are settled decisions, repeatedly corrected in the past. Honour them in every feature rather than reinventing per screen; do not re-litigate them.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lovelace-co/lovelace](https://github.com/lovelace-co/lovelace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
