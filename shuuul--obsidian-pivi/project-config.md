---
trigger: always_on
description: Welcome to the **Pivi** developer reference guide. This document is the **operational** entry point for build, test, lint, release, project glossary, quality gates, and repo-wide seam rules.
---

# Pivi Developer Guide

Welcome to the **Pivi** developer reference guide. This document is the **operational** entry point for build, test, lint, release, project glossary, quality gates, and repo-wide seam rules.

---

## 📚 Project guidance

Pivi keeps the new-developer architecture, technology rationale, end-to-end flows, and contribution routes in the [`docs/` handbook](docs/README.md). Durable operational guidance remains in layered `AGENTS.md` files: root guidance covers repo-wide build, test, release, and seam rules; package-local files cover package purpose, public entrypoints, boundaries, and verification notes.

For README architecture / workflow diagrams, prefer fenced Mermaid diagrams (` ```mermaid `) because GitHub renders them natively.

| Layer | Location | When to update |
|-------|----------|----------------|
| Developer handbook | `docs/README.md`, `docs/[0-9][0-9]-*.md` | User-visible behavior, end-to-end flows, public interfaces, persistence/configuration, boundaries, technology choices, development/release routes, or roadmap changes |
| Long-running specs | `specs/README.md`, `specs/NNN-*.md` | Multi-agent work breakdown, decisions, handoffs, verification, and completion tracking |
| Repo operations | `AGENTS.md` | Build/test/release/spec workflow changes |
| Package contracts | `packages/*/AGENTS.md` | Package entrypoints, dependency boundaries, or gotchas change |
| Feature maps | `src/ui/AGENTS.md`, `src/ui/chat/AGENTS.md`, `src/ui/chat/rendering/AGENTS.md`, `src/ui/shared/AGENTS.md`, `src/app/AGENTS.md`, `packages/pivi-react/AGENTS.md`, `packages/pivi-react/src/i18n/AGENTS.md`, `packages/pivi-react/styles/AGENTS.md`, `src/ui/chat/ui/file-context/AGENTS.md`, `src/ui/chat/ui/file-context/state/AGENTS.md`, `src/ui/chat/ui/file-context/view/AGENTS.md` | Local UI/runtime flow or seam rules change |
| Glossary/overview | `AGENTS.md` | Project identity or canonical terminology changes |
| Releases | GitHub Releases / generated `CHANGELOG.md` | User-visible release history |

**Workflow**

1. Explore in Obsidian / Heptabase (optional).
2. For long-running or multi-agent work, create and index a tracked spec from [`specs/000-template.md`](specs/000-template.md) before delegating work. Keep its decisions, workstreams, verification, and handoffs current.
3. Implement in the owning package or app area.
4. Update the closest `AGENTS.md` whenever code invalidates its map, seam rules, terminology, or gotchas. Start with the directory you changed and walk upward until guidance remains accurate.
5. Update the relevant numbered developer document in the same change whenever code changes user-visible behavior, an end-to-end flow, a public interface/type, configuration or persistence, a package boundary, a technology choice, development/release commands, or roadmap status. Keep package-local operational invariants in the owning `AGENTS.md` and link to the handbook for narrative detail.
6. Before completing and archiving a spec, synchronize every durable conclusion into the relevant numbered docs and the nearest affected `AGENTS.md` files, walking upward until the guidance remains accurate.
7. Before committing, review the staged diff and confirm the active spec, handbook, and nearest `AGENTS.md` files still describe it. Behavior-preserving internal refactors and test-only changes do not require documentation churn unless they invalidate a path, command, map, or verification rule.
8. Let release-please generate release notes and `CHANGELOG.md` from Conventional Commits in release PRs; keep README version changes in `node scripts/sync-version.js`, not release-please README markers.

For UI or runtime changes that the user needs to inspect inside Obsidian, run the project build and reload the plugin before handing back control. The normal path is `npm run build` followed by `obsidian plugin:reload id=pivi`, unless the user explicitly asks not to reload or the Obsidian CLI is unavailable.

**PR checklist** (include in description when applicable):

```markdown
Related guidance:
- Package: packages/<name>/AGENTS.md
- Local: <nearest>/AGENTS.md
```

| Change size | Documentation |
|-------------|----------------|
| Small fix | Code comment only when the why is non-obvious |
| Medium feature | Active spec when work is long-running/multi-agent; relevant numbered developer doc plus owning package/local `AGENTS.md` when its map or rules change |
| Architecture / framework | Active spec, developer handbook, root guidance, and affected package/local `AGENTS.md` |
| Stable module API | Relevant developer doc plus owning package `AGENTS.md` |
| User-visible UI text | Always `packages/pivi-react/src/i18n/` in the **same commit** (see Coding Standards) |

---

## 🤖 Agent skills

This repo does not track repo-local agent skills. Keep developer narrative in `docs/` and operational project guidance in this file and package/local `AGENTS.md` files; runtime vault skills live under each vault's `.pivi/skills/` directory.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuuul/obsidian-pivi](https://github.com/shuuul/obsidian-pivi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
