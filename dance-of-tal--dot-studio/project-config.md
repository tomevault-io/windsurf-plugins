---
trigger: always_on
description: `studio` is the local editor for APM Studio.
---

# AGENTS.md

## Purpose

`studio` is the local editor for APM Studio.

APM Studio helps users manage AI coding assistant packages as agents, instructions, skills, teams, and workflows. The product model is:

1. Import: discover community source references, presets, and compatibility metadata.
2. Manage: edit local APM-backed packages in `packages/*`.
3. Run: execute local agents and teams inside Studio without exporting Studio-only model settings.
4. Export: sync selected local packages into assistant-specific target files through target sync.

Think about the codebase in this order:

1. `src/` renders and edits Studio state in the browser.
2. `shared/` defines contracts shared by client and server.
3. `server/routes/` exposes HTTP boundaries.
4. `server/services/` owns package storage, registry import, runtime prep, projections, sync, and API behavior.
5. `packages/<packageId>/apm.yml` is the canonical package source for new package content.
6. `.apm-studio/workspace.json` stores Studio-only workspace state such as canvas layout and local UI metadata.
7. `.opencode/` holds generated OpenCode-facing runtime artifacts.
8. OpenCode executes projected runtime artifacts outside the React app.

If you need the OpenCode source, use `<local-opencode-source>`.

If you need the upstream Microsoft APM reference source for Studio development, use `<local-apm-source>`.

The official Microsoft APM documentation is at `https://microsoft.github.io/apm/`.

The sibling `<local-apm-registry>` checkout is the Cloudflare Worker registry project. Treat it as a separate project from this Studio repo.

## Top-Level Structure

- `src/`
  - Frontend application.
  - Main UI, canvas, local Packages drawer, Import page, Export page, assistant chat, agent chat, team UI, Zustand store.
  - Treat this as the Studio interaction layer.
- `shared/`
  - Shared contracts and runtime-safe types.
  - Keep client/server protocol shapes here.
  - Do not put browser-only or server-only behavior here.
- `server/routes/`
  - HTTP boundary.
  - Thin route layer that validates/forwards requests into services.
- `server/services/`
  - Main backend behavior.
  - Studio workspace operations, APM package storage, Import discovery, GitHub import, chat/session orchestration, draft handling, target sync, runtime prep.
- `server/services/apm-package/`
  - APM manifest, lockfile, YAML, package paths, and repository helpers.
  - New canonical package content should flow through this boundary.
- `server/services/import/registry-service.ts`
  - Import catalog discovery used by APM Assistant package hints.
  - Local GitHub source import and conversion belongs in `server/services/apm-package/github-import.ts`.
- `server/services/apm-package/target-sync.ts`
  - Manual external assistant target sync boundary.
  - Target capability, CLI-first sync, temporary package assembly, and Studio fallback projection should stay behind this service family.
- `server/services/opencode-projection/`
  - Projection boundary from Studio packages into OpenCode-consumable runtime artifacts.
  - Studio-internal OpenCode runtime projection remains automatic.
- `server/services/team-runtime/`
  - Team runtime scheduling and collaboration internals.
  - Keep this as the Team runtime boundary; user-facing language should be Team or Workflow.
- `server/services/studio-assistant/`
  - Runtime-only APM Assistant projection and prompt/action layer.
- `packages/`
  - Local APM package source of truth for new package data.
  - Each package must remain APM-native: `apm.yml` plus its own `.apm/` primitive source tree.
  - Do not bypass APM package helpers when changing package content.
- `.apm-studio/`
  - Studio-only workspace state, drafts, caches, and UI metadata.
  - Do not store new canonical package content here.
- `.opencode/`
  - Generated/projected OpenCode workspace data and manifests.
  - Useful for debugging projection output.
  - Do not treat this as the main source of truth unless the task is explicitly about projection artifacts.
- `doc/`
  - Detailed architectural and behavioral guides.
  - Read the relevant docs before making non-trivial runtime/session/assistant changes.
- `DESIGN.md`
  - Studio design system guide for humans and coding agents.
  - Read before adding or changing frontend UI.
  - Keep aligned with `src/tokens.css` and `src/primitives.css`.
- `public/`
  - Static assets served by the app.
- `client/`, `dist/`
  - Build outputs.
  - Prefer changing source files, not generated output.

## Import, Manage, Run, Export Boundary

The core package flow is:

`registry listing -> import adapter -> packages/<packageId>/apm.yml -> Manage edits -> Run in Studio or Export target sync -> assistant files`

- Import is discovery/import, not the package source of truth after import.
- The Packages drawer is local-only: show installed/draft packages and runtime primitives there, not registry Import search.
- Registry listings should reference GitHub sources plus import recipes, trust/index metadata, and presets. They should not store user workspace state, private config, generated assistant files, or edited local package content.
- Manage uses local APM packages under `packages/*` as the canonical editable source.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dance-of-tal/dot-studio](https://github.com/dance-of-tal/dot-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
