---
trigger: always_on
description: `CLAUDE.md` is a symlink to this file (`AGENTS.md`). Edit `AGENTS.md` directly.
---

# Commonplace

`CLAUDE.md` is a symlink to this file (`AGENTS.md`). Edit `AGENTS.md` directly.

A framework for building agent-operated knowledge bases. This repo contains the methodology, type definitions, writing conventions, instructions and skills, and the Python commands that get installed into projects.

The commonplace repo is itself a knowledge base — it uses its own knowledge system to document the methodology for building knowledge bases. There is no separation between "user content" and "methodology" here; the methodology IS the content.

## KB Goals

### Purpose

This KB accumulates methodology for building agent-operated knowledge bases — the theory, design patterns, and operational conventions that make KBs effective. It supports agents and maintainers making decisions about KB architecture, type systems, writing quality, context engineering, and knowledge organization.

### Domain

Agent-operated knowledge base methodology: how to structure, write, connect, validate, review, and maintain knowledge artifacts for consumption by LLM agents. Adjacent topics (learning theory, cognitive science, software architecture) are in scope only where they directly inform KB design decisions.

### Include

- Design decisions about KB methodology (type systems, linking, indexing, review)
- Context engineering theory (distillation, constraining, codification)
- Operational patterns (writing workflows, validation, maintenance)
- Evaluations and comparisons with external knowledge systems

### Exclude

- Application-specific KB content (belongs in consuming projects)
- General software engineering unless it informs KB methodology
- Raw logs without analysis (use `kb/log.md`)

### Quality bar

A design insight is worth a note when it changes how someone would build or operate a KB. Observations about what works are worth a log entry on first occurrence and a note when the mechanism is understood. Pure pattern-recording without explanation belongs in a log entry, not a note.

## Key Indexes

- `kb/notes/tags-index.md` — top-level navigation hub: tag indexes, foundations, evaluation, gaps
- `kb/notes/links-index.md` — linking methodology: semantics, navigation, contracts
- `kb/agent-memory-systems/README.md` — curated index of external agent-memory/knowledge systems
- `kb/agent-memory-systems/dir-index.md` — auto-generated directory listing for agent-memory-system coverage
- `kb/notes/dir-index.md` — auto-generated directory listing (rebuild with `commonplace-refresh-indexes`)
- `kb/reference/README.md` — shipped-system documentation entry point: architecture, type system, operator guide, and ADR navigation
- `kb/reference/navigation.md` — how agents navigate the KB with `rg`, titles/descriptions, indexes, links, connect reports, and future search layers
- `kb/reference/adr/` — architecture decision records for the shipped commonplace system
- `kb/reference/link-vocabulary.md` — label catalogue and authoring guidance for `COLLECTION.md` authors (consult when revising outbound rules)
- `kb/sources/dir-index.md` — auto-generated source listing

## Vocabulary

Terms used in this KB with specific meanings. On first mention in a note, gloss and link: `[distillation](./definitions/distillation.md) (directed context compression)`.

- **Context engineering** — the architecture and machinery for getting the right knowledge into a bounded context at the right time. Includes routing, loading, scoping, and maintenance. See `kb/notes/definitions/context-engineering.md`.
- **Distillation** — compression viewed as learning: goal-oriented compression whose purpose is the capacity change it produces in the consumer. KB application is directed context compression — the main operation context engineering performs. ML knowledge distillation (Hinton) is a sibling instance in a different substrate. See `kb/notes/definitions/distillation.md`.
- **Constraining** — narrowing the interpretation space of an artifact (less generality, more reliability/speed/cost). Orthogonal to distillation. See `kb/notes/definitions/constraining.md`.
- **Codification** — committing a procedure to a symbolic medium (natural language → code). The far end of constraining. See `kb/notes/definitions/codification.md`.
- **Register** — one of three content modes (theoretical, descriptive, prescriptive) that determines a collection's quality goal, title conventions, and linking rules. See `kb/notes/definitions/register.md`.
- **Workshop** — a named workspace for temporal, work-in-flight documents. Lives in `kb/work/<workshop-name>/`. Value is consumed rather than accumulated — workshop artifacts have lifecycles and expiration; they produce library artifacts (notes, ADRs) when done. Contrast with the library layer (notes, indexes) where value accumulates over time. See `kb/notes/a-functioning-kb-needs-a-workshop-layer-not-just-a-library.md`.

## Development

- **Use `python3`** for stdlib-only throwaway tooling. Commonplace runtime code should live in the Python package and be invoked through `commonplace-*` commands. Use `uv run` for tests and optional-dependency workflows (`pytest`, `xdk`, MkDocs, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zby/commonplace](https://github.com/zby/commonplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
