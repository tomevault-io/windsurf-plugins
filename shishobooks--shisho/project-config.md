---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Codex, Pi, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, Codex, Pi, etc.) when working with code in this repository.

## Subagent Instructions

**When dispatching subagents (for implementation, code review, spec review, or any other task), always include this instruction in the prompt:**

> Check the project's root AGENTS.md and any relevant subdirectory AGENTS.md files for rules that apply to your work. These contain critical project conventions, gotchas, and requirements (e.g., docs update requirements, testing conventions, naming rules). Violations of these rules are review failures.

Subdirectory AGENTS.md files are loaded automatically when working on files in that directory, but cross-cutting rules (like "update website docs when changing user-facing behavior") live in this root file and are easy to overlook if not explicitly checked.

## Important Notes

**When `mise tygo` prints "skipping, outputs are up-to-date", this is NORMAL.** It means the generated types are already up-to-date (mise checks source/output timestamps). Do not treat this as an error. The user often has `mise start` running in another session which runs tygo automatically via air, but you should still run `mise tygo` yourself (especially in worktrees where `mise start` may not be running).

**Keep AGENTS.md files up to date.** Subdirectory `AGENTS.md` files document patterns, conventions, and gotchas for each area of the codebase. When you make changes that affect what's documented, such as adding new patterns, changing APIs, renaming fields, or adding new conventions, update the relevant `AGENTS.md` to reflect the new state. Outdated documentation is worse than no documentation.

- **Domain-specific** (patterns, gotchas, conventions for a specific area) → Update or add to the relevant `AGENTS.md` in the subdirectory (e.g., `pkg/epub/AGENTS.md`)
- **Project-wide** (general conventions, critical gotchas, workflow rules) → Update or add to this file (AGENTS.md)

Examples of things to record: discovered gotchas, naming conventions, architectural decisions, common mistakes, integration patterns, edge cases.

## Subdirectory AGENTS.md Files

Project-specific conventions are documented in `AGENTS.md` files within each subdirectory. These are automatically loaded when working on files in that directory.

| Location | Covers |
|----------|--------|
| `pkg/AGENTS.md` | Go backend: Echo handlers, Bun ORM, workers |
| `app/AGENTS.md` | React frontend: Tanstack Query, components, UI patterns |
| `app/components/layout/AGENTS.md` | Shared layout primitives: Sidebar, UserMenu, top-nav class constants |
| `pkg/plugins/AGENTS.md` | Plugin system: Goja runtime, hooks, host APIs, manifests |
| `pkg/epub/AGENTS.md` | EPUB format: OPF, Dublin Core, parsing/generation |
| `pkg/cbz/AGENTS.md` | CBZ format: ComicInfo.xml, creator roles, chapter detection |
| `pkg/kepub/AGENTS.md` | KePub format: koboSpan wrapping, CBZ-to-KePub conversion |
| `pkg/mp4/AGENTS.md` | M4B format: iTunes atoms, chapters, narrator fallback |
| `pkg/pdf/AGENTS.md` | PDF format: info dict metadata, pdfcpu thread safety |
| `pkg/pdfpages/AGENTS.md` | PDF page cache: render/cache PDF pages as JPEG, thread safety, config |
| `pkg/events/AGENTS.md` | SSE: event broker, streaming handler, event types |
| `pkg/audnexus/AGENTS.md` | Audnexus chapter lookup: cached HTTP client, typed error codes, the M4B chapter route |
| `website/AGENTS.md` | Docs site: Docusaurus, versioning, deployment |
| `e2e/AGENTS.md` | E2E testing: Playwright, per-browser isolation, fixtures |
| `tools/gotestsplit/AGENTS.md` | Timing-aware Go test sharding: cache strategy, picking shard count, recalibration playbook |

## Utility Skills

These workflow-based skills (in `.claude/skills/`) are invoked on demand:

| Skill | Invoke When |
|-------|-------------|
| `favicon` | Creating or updating favicon, app icons, PWA icons |
| `splash` | Creating or updating the README splash image |
| `metadata-field` | Adding, removing, or significantly modifying a metadata field on books or files |

## Critical Gotchas

These are common mistakes that cause bugs. Most are summarized here in a line and documented in detail in `pkg/AGENTS.md`, `app/AGENTS.md`, or `pkg/plugins/AGENTS.md`; the self password reset rule lives only here.

### Backend

**Request binding must use structs.** The custom binder uses mold and validator, which only work with structs, so never bind directly to a slice or array. See "Request binding must use structs" under API Conventions in `pkg/AGENTS.md`.

**`CoverImageFilename` stores the filename only**, never a full path; use `filepath.Base()` when updating it. See "Cover Image System" in `pkg/AGENTS.md`.

**JSON field naming is `snake_case`**, except the plugin manifest and repository-index passthrough fields. See "API Conventions" in `pkg/AGENTS.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shishobooks/shisho](https://github.com/shishobooks/shisho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
