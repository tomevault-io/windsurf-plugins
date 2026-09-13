---
trigger: always_on
description: - Always commit changes after all edits are done. Do not leave uncommitted changes at the end of a task.
---

# AGENTS.md

## **IMPORTANT Global Instructions for Agents:**

- Always commit changes after all edits are done. Do not leave uncommitted changes at the end of a task.
- This app has no real users or real data yet. Prefer long-term architectural correctness over short-term compatibility. Breaking changes, schema migrations, and large refactors are acceptable when they make the product model simpler and more coherent.
- For frontend design, prioritize an elegant, information-dense layout with minimal logical/visual redundancy and shallow nesting.
- Keep implementation notes, docs, changelog entries, commit messages, and handoff summaries product-native: describe what Nodex does and why, without surfacing private provenance, comparative targets, or reconstruction details unless the user explicitly asks for research notes.
- Do not read repository contents via web crawling from `raw.githubusercontent.com` because it is not stable for agent workflows. For remote repository inspection, clone the repository into a temporary local directory and read files from the local clone instead.
- DO NOT write tests that only assert a source file contains a string (source-string tests); that is redundant with the implementation and does not validate behavior.
- Read [official doc of codex-app-server](https://developers.openai.com/codex/app-server.md) when dealing with codex-app-server.

### Desktop UI inspection boundary

- Inspect Nodex through a seeded development instance, Playwright, or an explicitly identified development CDP target.
- Treat every already-running Nodex window as the user's production app unless its current-worktree, disposable development Profile is proven; its title, bundle identifier, or running state is not proof.
- `computer-use` is opt-in: use it only when the user explicitly requests it for the current task, or approves a proposal that names the target instance and purpose. Inspection, testing, and screenshot requests are not implicit permission, and approval does not carry between tasks.

## Agent skills

### Domain docs

Nodex uses a single-context layout with one root `CONTEXT.md` and system-wide ADRs under `docs/adr/`.
See `docs/agents/domain.md`.

## Project Overview

Nodex is a local-first, block-based agent orchestrator.
It ships as an Electron desktop app plus a CLI/HTTP API backed by SQLite.

## Setup Commands

- Install deps: `vp install`
- Dev app: `vp run dev`
- Dev Storybook: `vp run dev:storybook` (defaults to port 6006; override an occupied port with `STORYBOOK_PORT=6007 vp run dev:storybook`)
- Build: `vp run build`
- Package installers: `vp run package`
- Unified check: `vp run check`
- Semantic check without formatting: `vp run typecheck`
- Full semantic aliases: `vp run check:semantic`, `vp run typecheck`, `vp run lint` (shared cache)
- Format check: `vp run fmt:check`
- Standard tests: `vp run test`
- Source gate: `vp run verify:source` (`vp run test:all` is an alias)
- macOS runtime gate: `vp run verify:runtime:mac`
- Signed dual-architecture gate: GitHub `Distribution Rehearsal`

## Runtime and Tooling

- Package manager: pnpm
- Development runtime: Node 24.15.0
- Native addon: `node-pty` is rebuilt for Electron by `postinstall`; host Node and Electron have different ABIs even when they report the same Node version.
- Engineering control plane: Vite+ with TypeScript 7, Oxlint, Oxfmt, and Vitest; Playwright for Chromium and Electron E2E
- Language: TypeScript (`strict` mode)
- Desktop shell: Electron + electron-vite
- Frontend: React 19 + Tailwind + Radix + BlockNote/Prosemirror
- Backend: detached Rust Core (`rusqlite` + Yrs) with a Hono desktop adapter

### Effect 4 retrieval

Before editing Effect code, read `docs/adr/0047-effect-control-plane-and-runtime-boundaries.md`
and the nearest Nodex implementation, then read the installed version's
`node_modules/effect/AGENTS.md`. Search `node_modules/effect/ai-docs/src` for usage guidance
and the relevant package's `src` directory for API or implementation details. Nodex's
architecture boundaries override upstream examples. Treat Context7, web documentation, and
Effect `main` as secondary sources that must be verified against the installed version; clone
that version's exact upstream tag only when its tests are needed to resolve deeper semantics.

## Code Style

- **DRY**: Always keep code DRY. Extract shared hooks, helpers, and patterns instead of duplicating.
- **Tailwind over custom CSS**: Use Tailwind utility classes. Avoid inflating `globals.css` with new custom class rules.
- Before defining any new protocol-facing type, first check `packages/codex-app-server-protocol/src/v2`.
- Treat `packages/codex-app-server-protocol/src/v2` as the source of truth for Codex app server request/response/notification/thread shapes.
- Prefer importing protocol types directly, or re-exporting them as aliases when a local name is needed.
- Do not hand-write parallel protocol field definitions in `src/shared/types.ts` unless the local type is intentionally a derived or view-model shape.
- Keep data validation at boundaries (`src/main/http-server.ts`, generated Core contracts, and shared transport-neutral validators such as `src/shared/page-input-validation.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junyudev/nodex](https://github.com/junyudev/nodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
