---
trigger: always_on
description: Documentation is the most important part of this project.
---

# AGENTS

## Documentation First

Documentation is the most important part of this project.

Treat every `AGENTS.md` file as part of the runtime contract, not as optional notes. Poor documentation causes agent behavior drift, architecture drift, and bad changes in the wrong layer.

This repository now uses a documentation hierarchy:

- `/AGENTS.md` owns repo-wide rules, documentation policy, and top-level architecture
- the five core docs are `/AGENTS.md`, `/app/AGENTS.md`, `/server/AGENTS.md`, `/commands/AGENTS.md`, and `/packaging/AGENTS.md`
- deeper `AGENTS.md` files inside `app/` and `server/` own the concrete implementation contracts for the module or subsystem in their subtree
- the closer the doc is to the code, the more technical and specific it should be
- the higher the doc is in the tree, the more it should focus on principles, ownership, stable contracts, and architecture

Always update the relevant docs in the same session as the code change:

- update the closest owning `AGENTS.md` for the files you changed
- reflect stable architecture, workflow, and agent-facing behavior changes into the supplemental docs under `app/L0/_all/mod/_core/documentation/docs/`
- update parent docs too when the higher-level contract, ownership boundary, architecture, or workflow changed
- keep higher-level docs abstract where appropriate and push implementation detail down into local docs
- keep lower-level docs concrete, explicit, and practical
- remove stale or contradictory documentation immediately
- keep `README.md` as the public product source of truth for the project pitch, quick starts, community links, release links, and the high-level documentation map
- do not let `README.md` become a competing implementation contract; durable architecture and workflow contracts belong in `AGENTS.md`, while broader agent-facing narrative docs belong in `app/L0/_all/mod/_core/documentation/docs/`

Documentation depth model:

- level 0 repo doc: mission, top-level architecture, cross-cutting rules, and the documentation policy for the whole tree
- level 1 core domain docs: `app/`, `server/`, `commands/`, and `packaging/` architecture, plus the template for their immediate child docs
- level 2 subsystem or module docs: one major area, its owned files, its stable contracts, and the template for any deeper docs in that subtree
- level 3 leaf docs: one concrete surface, feature, view, or service contract with exact implementation guidance
- if a level 2 or level 3 doc later gains children, it stops being a leaf and must add its own `Documentation Hierarchy` section before those child docs land

Documentation shape rules:

- sibling docs at the same depth should use the same section order unless a domain-specific contract truly needs one extra section
- every `AGENTS.md` should answer, in this order when practical: what this scope owns, which files or surfaces it owns, which stable contracts it enforces, how child docs divide the remaining detail, and what changes require doc updates
- parent docs explain boundaries, ownership maps, and stable seams; child docs explain concrete file-level behavior, state, styles, assets, and API usage
- do not split one ownership boundary across multiple ad hoc notes when one owning `AGENTS.md` can explain the links between code, styles, state, assets, and APIs more clearly

## Introduction

Space Agent is a browser-first AI agent runtime.

The browser app is the primary runtime. The Node.js side exists as thin infrastructure around it for:

- outbound fetch proxying when the browser would otherwise hit CORS limits
- server-owned APIs and other narrow infrastructure contracts
- local development and optional desktop hosting

Default architecture rule:

- prefer frontend changes over backend changes
- treat backend edits as exceptional, not routine
- only move work into `server/`, `commands/`, or `packaging/` when the behavior must be backend-owned for security, integrity, multi-user isolation, or runtime stability and cannot be safely enforced in the browser
- if backend work is needed and the user did not explicitly ask for it, stop and ask for permission before editing backend files
- when asking for that permission, explain that backend work is non-standard for this project, describe the risk that makes frontend-only work insufficient, and state the narrow backend change you need

Implement only what the user explicitly asked for. Do not invent new features, policies, cleanup behavior, or product changes on your own. If a request would require a new behavior or policy that the user did not ask for, stop and ask first.

The five core documentation files remain the project's primary instruction set:

- `/AGENTS.md`
- `/app/AGENTS.md`
- `/server/AGENTS.md`
- `/commands/AGENTS.md`
- `/packaging/AGENTS.md`

## AGENTS File Index

This root file must keep an exhaustive index of every other repo `AGENTS.md` path so the full contract map is visible without extra filesystem discovery. Update this index whenever an `AGENTS.md` file is added, removed, moved, or renamed.

App docs:
- `/app/AGENTS.md`
- `/app/L0/_admin/mod/_core/overlay_agent/AGENTS.md`
- `/app/L0/_all/mod/_core/admin/AGENTS.md`
- `/app/L0/_all/mod/_core/admin/views/agent/AGENTS.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent0ai/space-agent](https://github.com/agent0ai/space-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
