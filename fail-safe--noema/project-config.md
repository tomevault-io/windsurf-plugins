---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Also read .claude/layout.md when it exists.

## Project

**Noema** — "The intentional memory layer for your AI agents."

Noema         → The project
Cortex        → The collection / instance
Trace         → A memory or piece of knowledge

## Etymology

From Husserl's phenomenology: Noema is the intentional content of a mental act — the "what" that a thought is about, as it appears to consciousness.

The paired concept is Noesis — the act of thinking itself. Noema is what is thought; Noesis is the thinking of it. Traces are the collective units [memory entry] comprised of a markdown doc + corresponding DB rows. The Cortex is the collection of Traces.

## Commands

```bash
go build ./cmd/noema   # build the binary
go run ./cmd/noema     # run without building
go build ./...         # build all packages (CI check)
go test ./...          # run all tests
go vet ./...           # static analysis
```

## Branch Strategy

- `main` is the stable/release branch
- Use a `feature` or `bug` branch for active work; PRs go feature/bug-branch → `next`
- Use a `next` branch for staging/testing upcoming (beta) work; PRs go `next` → `main`

## Tech Stack

Super simple, super basic. Intentionally lightweight. Transparently open.

**Language:** Go (v1). Revisit Rust if MCP server concurrency demands it.
- SQLite: `modernc.org/sqlite` (pure-Go, no CGo required)
- TUI: Charm / Bubble Tea
- Search: SQLite FTS5 — no semantic/vector search in v1

Markdown files are the content holders of data. The SQLite DB is a basic schema to hold pieces of metadata about the markdown files (location, author, type, tag(s), ...) in order to provide mapping, related contexts, search performance.

The collective unit [memory entry] comprised of markdown doc + corresponding DB rows will be known as a 'Trace'.

A Trace shall have a single markdown doc, though the doc may be as small or large as needed. Traces may be categorized as one of several types based on intent. They may be tagged in such a way as to allow for distinct Traces to be formed into logical groupings or patterns as the data grows.

A collection or instance containing Traces will be known as a 'Cortex'.

### Trace Structure

Every Trace is a markdown file with YAML frontmatter followed by free-form content:

```markdown
---
id: 20260329-why-we-chose-go
title: Why we chose Go
type: decision
author: research-agent-1
tags: [go, language, architecture]
derived_from: [20260328-language-candidates]
origin: research-cortex
created: 2026-03-29T14:23:00Z
updated: 2026-03-29T14:23:00Z
---

Body content here.
```

All frontmatter fields are indexed in the DB. `author` is a free-form string — it can be a human username, an agent name, or omitted. Multi-agent systems use it to identify which peer wrote a given Trace.

`derived_from` is an optional list of trace IDs this Trace was derived from — it builds a lineage graph queryable via `trace_lineage`. `origin` is the name of the Cortex that produced the Trace; it is set automatically on creation and used by federation to attribute remote Traces. Both fields use `omitempty`, so older Traces without them parse unchanged.

### Trace Types

Every Trace has exactly one type:

| Type | Meaning |
|---|---|
| `fact` | A discrete thing that is true |
| `decision` | A choice made and why |
| `preference` | A behavioral or stylistic lean |
| `context` | Situational background |
| `skill` | A learned capability or procedure |
| `intent` | Something that needs to happen (pickup is autonomous) |
| `observation` | Something witnessed but not yet verified |
| `note` | Anything else |
| `divergence` | A concurrent edit conflict, auto-created by federation sync |

### Cortex Storage Layout

Each Cortex is a named directory the user manages. Layout:

```
<cortex-dir>/
  cortex.md         ← Cortex manifest (see below)
  archive/
    traces/         ← Archived traces, same format as active traces
  db/
    noema.db        ← SQLite DB: metadata, tags, FTS5 index
  traces/
    20260329-why-we-chose-go.md
    20260329-another-trace.md
  trash/
    traces/         ← Removed traces, kept for 30 days [user-configurable]
```

Trace filenames follow the pattern `YYYYMMDD-slugified-title.md` (ISO 8601). The markdown files are the source of truth for content; the DB is the index.

**`cortex.md` manifest** — a markdown file with YAML frontmatter. The
YAML block holds the manifest (minimal — not a config file); any prose
below the closing fence is preserved verbatim on write, so users can
keep free-form notes about the cortex alongside its metadata.

```markdown
---
id: 01JQ3Z4XKP8VY6H7B9W2R5T8MN
name: my-cortex
purpose: "Primary memory for the research agent cluster"
owner: mark
created: 2026-03-29
version: 2
---

Optional free-form notes about this cortex.
```

For back-compat, `ReadManifest` also accepts legacy bare-YAML files
(no `---` fences) written by older binaries; the next `WriteManifest`
silently upgrades them to the framed form.

### Archiving

Archiving is non-destructive and fully reversible:

- `noema archive <id>` — moves the markdown file to `archive/traces/`, sets `archived_at` timestamp in DB

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fail-Safe/Noema](https://github.com/Fail-Safe/Noema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
