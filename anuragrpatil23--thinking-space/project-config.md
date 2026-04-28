---
trigger: always_on
description: Required operating contract for any coding agent working in `Thinking Space`.
---

# AGENTS.md

Required operating contract for any coding agent working in `Thinking Space`.

If this file conflicts with assumptions, follow this file + `DEVELOPMENT.md`.

## Mission
Build one product that is intentionally all three of these from the ground up:

1. Thinking space for individuals
- Audience: knowledge workers, researchers, writers, founders
- Value: fast, local, hierarchical thinking (`Programs -> Epics -> Ideas -> Thoughts`)
- Entry: "I need a better way to organize my thoughts"

2. Place where humans and AI work together
- Audience: AI-savvy users frustrated by disconnected tools
- Value: thinking and AI assistance in the same contextual workspace
- Entry: "AI tools are useful but disconnected from where I actually think"

3. AI agent management space for humans
- Audience: power users, developers, multi-agent operators
- Value: manage agents, track runs/work, integrate output with human thinking
- Entry: "I am running AI agents but have nowhere to manage them alongside my own thoughts"

These are architecture requirements, not optional positioning variants.

## Strategy Rules
- Do not design isolated feature silos for only one pillar.
- Prefer shared primitives that strengthen all three pillars.
- Any major change must state pillar impact before implementation.

## Working Style (Derived from CLAUDE.md)
- Think from first principles, then map decisions to concrete code tradeoffs.
- Stay concise and direct; avoid filler and vague recommendations.
- Challenge weak assumptions politely and propose better alternatives.
- Optimize for practical progress, not theoretical architecture purity.

## CLAUDE.md and AGENTS.md
- `CLAUDE.md` is Claude Code's native file for project-specific instructions.
- `AGENTS.md` is the cross-tool open-standard contract.
- Both should contain consistent onboarding, architecture constraints, and execution priorities.
- If Claude learns useful project knowledge, Claude must manually update `CLAUDE.md` and synchronize durable items into `AGENTS.md` plus organizer principles/decision records.

## Phase Order
Use `DEVELOPMENT.md` as source of truth for implementation phases and detailed architecture.

Current status (v2.5): Phases 0–5, Agent Capability Transport, EPIC-3, Embedded Terminal, Live Source Mode, notebook workspace upgrades, and native iPhone shell/chrome work are all complete.

Upcoming:
- EPIC-5: AI Actions Everywhere
- EPIC-6: Optional Remote/Agent Backends (later)

If sequence changes, update `DEVELOPMENT.md` first, then align active organizer plan/task nodes.

## Locked Technical Decisions
1. Electron-first runtime for near-term milestones.
2. **YAML frontmatter in Markdown files** as source of truth for all node metadata and hierarchy.
3. **IndexedDB (Dexie.js)** as rebuildable in-browser cache. NOT a source of truth.
4. **No SQLite / native DB** — previous SQLite plan is superseded.
5. **No backend required** for core features (hierarchy, editing, AI).
6. **Folders are arbitrary** — hierarchy is metadata-driven via YAML `parent` fields.
7. Lexical related retrieval first via IndexedDB full-text search.
8. Local-only extensions first; no early remote code execution.
9. AI local-first: Ollama (Electron) or WASM LLM (web/PWA).
10. Extension platform rollout is feature-flagged (`extension_host_enabled`, `extension_builder_enabled`) and defaults to disabled until explicitly enabled.
11. **Embedded Terminal**: xterm.js + node-pty (same stack as VS Code). `TerminalPage.tsx` mounts all tabs simultaneously (`visibility:hidden`) so shells survive tab switches. PTY routing uses `webContentsId` stored in `ptyManagerBlock.ts`.
12. **Live Source Mode**: source code ships inside the DMG (`Resources/source/`), extracted to `userData/source/` on first launch. Power users can point at their own git repo. Vite dev server is spawned from `viteServerBlock.ts`; renderer switches to `http://127.0.0.1:{port}` without restart. Rebuild via `viteRebuildBlock.ts` → detached swap script replaces the running `.app` and relaunches.

## Architecture Reference
Full YAML schema and architecture details: `docs/ADR-004-YAML-Architecture.md`

## Architecture Guardrails
- Keep markdown files with YAML frontmatter as portable source-of-truth content.
- Hierarchy is defined by YAML `parent`/`type`/`level` fields, NOT folder structure.
- IndexedDB is a pure cache layer — can be rebuilt from YAML files at any time.
- Standardize markdown view/edit through one shared orchestrator (`frontend/src/components/orchestrators/MarkdownViewerOrch.tsx`); do not add page-local markdown edit modals.
- Reparent by updating YAML `parent` fields in affected files + syncing IndexedDB.
- Add conflict-safe saves for thought editing (`mtime`/hash checks).
- Avoid destructive migrations without rollback/recovery path.
- No backend dependency for core features.

## Code Design Philosophy
- Use lego blocks: small reusable primitives for UI, hooks, and services.
- Use orchestrators: page/feature containers that compose primitives and own flow/state wiring.
- Keep primitives generic and prop-driven; avoid feature-specific branching inside shared components.
- Keep data loading, derived selectors, and orchestration handlers in orchestrators.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anuragrpatil23/Thinking-Space](https://github.com/anuragrpatil23/Thinking-Space) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
