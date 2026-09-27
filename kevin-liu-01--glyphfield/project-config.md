---
trigger: always_on
description: > Pointer index for agents working in this repo. Keep this file lean: load linked
---

# glyphfield - Agent Index

> Pointer index for agents working in this repo. Keep this file lean: load linked
> files on demand, prune no-op instructions, and keep generated facts inside
> `agent-docs:auto` blocks.

## Overview

Glyphfield is a local-first brand system and production Studio. One `BrandIdentity`
feeds interactive tools for identity, brand applications, moodboards, books,
motion, Lottie, layered Design Lab compositions, foundations, templates, and
components. The same product is exposed to agents through discovery/catalog
routes, deterministic HTTP generation, processed Markdown docs, and the
programmatic Studio Browser API.

Start user-facing work by deciding which contract owns it:

- Deterministic data/SVG: `src/app/api/**` and `src/lib/agentGeneration.ts`.
- Authentic Canvas/WebGL/local-file output: the Studio component and
  `src/lib/studioAutomation.ts`.
- Portable layered composition: `src/lib/canvasDocument.ts` plus the host adapter.
- Product guidance: `content/docs/**`, `/llms.txt`, and generated `/llms-full.txt`.

## Product usage skills

When the task is to use Glyphfield rather than modify its implementation, load the
smallest matching checked-in Agent Skill:

- [`skills/glyphfield-create/SKILL.md`](skills/glyphfield-create/SKILL.md) — layered
  Design Lab composition and saved-design work.
- [`skills/glyphfield-api/SKILL.md`](skills/glyphfield-api/SKILL.md) — deterministic
  HTTP discovery and generation.
- [`skills/glyphfield-studio/SKILL.md`](skills/glyphfield-studio/SKILL.md) — live
  Browser API operation, source round trips, and local files.
- [`skills/glyphfield-export/SKILL.md`](skills/glyphfield-export/SKILL.md) — still
  and motion export verification.

Combine skills only when the requested workflow crosses those boundaries. These
packages describe product operation; the root and directory `SKILL.md` files
describe repository contribution.

## Architecture Pointers

- `src/components/StudioApp.tsx` — projects, tabs, navigation, active identity/tool.
- `src/lib/studioCatalog.ts` — public navigable tool IDs, names, categories, search.
- `src/components/StudioToolWorkspace.tsx` — maps public tools to editors.
- `src/components/ShaderLabStudio.tsx` — Design Lab composition, materials, saved
  designs, still/motion export, and its Browser API adapter.
- `src/components/AnimationStudio.tsx` — frame/timing/audio motion workspace.
- `src/lib/canvasDocument.ts` — portable scene graph and mutation/history model.
- `src/lib/designLabDocument.ts` — Design Lab ↔ CanvasDocument adapter.
- `src/lib/designWorkspace.ts` + `CONTEXT.md` — open canvas ownership, world-space placement, and artboard/output vocabulary.
- `src/lib/agentApi.ts`, `agentCatalog.ts`, `agentGeneration.ts` — public agent
  manifest, catalogs, validation, generation, and examples.
- `src/lib/studioAutomation.ts` — `window.glyphfield.studio` runtime contract.
- `src/lib/studioAgentCapabilities.ts` — canonical per-tool source, export, HTTP,
  and Browser action capability map shared by runtime discovery and docs.
- `content/docs/**` + `src/components/DocsMdx.tsx` — human and machine docs source.
- `public/llms.txt` — concise agent router; `src/app/llms-full.txt/route.ts` emits the
  complete processed docs corpus.

## Stack
<!-- agent-docs:auto:stack start -->
- **Name:** glyphfield
- **Package manager:** pnpm
- **Languages:** typescript
- **Framework:** next
<!-- agent-docs:auto:stack end -->

## Commands
<!-- agent-docs:auto:commands start -->
- Package scripts detected: 30. Use `package.json` as the exhaustive source.
- `pnpm run dev` - next dev --turbopack --port 3012
- `pnpm run build` - next build --turbopack
- `pnpm run test` - vitest run
- `pnpm run lint` - pnpm lint:fast && pnpm lint:cognitive
- `pnpm run typecheck` - fumadocs-mdx && tsc6 --noEmit
- `pnpm run agent-docs` - npx tsx scripts/run-agent-docs.ts
- Keep this block compact. Put full command catalogs in a generated command index, not in AGENTS.md.
<!-- agent-docs:auto:commands end -->

## Directory index
<!-- agent-docs:auto:dirmap start -->
| Directory | Skill | Purpose |
|---|---|---|
| `e2e/` | [`e2e/SKILL.md`](e2e/SKILL.md) | Isolated Chromium, WebKit, and Firefox Studio regression tests. |
| `scripts/` | [`scripts/SKILL.md`](scripts/SKILL.md) | Repository diagnostics and the canonical agent-docs forwarding shim. |
| `scripts/lib/` | [`scripts/lib/SKILL.md`](scripts/lib/SKILL.md) | Native Safari regression helpers and input-delivery diagnostics. |
| `src/app/` | [`src/app/SKILL.md`](src/app/SKILL.md) | Next routes, documentation shell, machine endpoints, and global styles. |
| `src/components/` | [`src/components/SKILL.md`](src/components/SKILL.md) | Studio editors, shared UI systems, and authentic browser renderers. |
| `src/hooks/` | [`src/hooks/SKILL.md`](src/hooks/SKILL.md) | Persistent, portable, and performance-sensitive React state lifecycles. |
| `src/lib/` | [`src/lib/SKILL.md`](src/lib/SKILL.md) | Product models, serializers, render/export helpers, and agent contracts. |
<!-- agent-docs:auto:dirmap end -->

## Repo graph sidecar (Graphify)
<!-- agent-docs:auto:repo-graph start -->
- Use Graphify for repo topology, path/explain/affected questions, PR risk, and unfamiliar codebase orientation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kevin-Liu-01/Glyphfield](https://github.com/Kevin-Liu-01/Glyphfield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
