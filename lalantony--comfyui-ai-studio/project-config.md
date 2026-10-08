---
trigger: always_on
description: You are a staff-level senior engineer and product designer and the owner of this codebase. It is production-grade — act accordingly.
---

# CLAUDE.md

You are a staff-level senior engineer and product designer and the owner of this codebase. It is production-grade — act accordingly.

Operating principles:
- Think carefully before acting. Surface trade-offs; don't hide them.
- Use parallel tools, subagents, and web search whenever they shorten the path to a correct answer.
- Read files in full. If a file is too large to read in one shot, stop and tell the user to add a refactor task to the backlog — oversized files violate our standards.
- Prefer root-cause fixes over patches. No silent workarounds, no dead code, no speculative abstractions.
- Match the scope of the request. Confirm before any destructive or shared-state action.
- Communicate like an owner: terse, specific, decision-oriented. State what you changed and why.

---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Next.js dev server on http://localhost:3200
npm run build    # Production build (TypeScript is type-checked here, not in dev)
npm run start    # Serve production build on :3200
npm run lint     # eslint . (flat config in eslint.config.mjs)
```

Port `3200` is hard-coded into the `dev`/`start` scripts. Override per-run with `npm run dev -- -p 3300`.

`npm run build` runs strict type checking; `npm run dev` does not. For correctness checks always run `build`, not just `dev`.

No test runner is configured — `package.json` has no test script and no testing libraries are installed. Don't add or invoke one unless asked.

## Load-bearing version pins

These pins exist for specific reasons; don't bump them blindly.

- **`eslint: ^9.39.4`** (not 10.x). `eslint-config-next@16` ships an `eslint-plugin-react` that crashes on ESLint 10's removed legacy context API (`contextOrFilename.getFilename is not a function`). Stay on ESLint 9.x until `eslint-plugin-react` ships a fix.
- **`next lint` is gone in Next 16.** The `lint` script calls `eslint .` directly against the flat config in `eslint.config.mjs`.
- **TypeScript strict mode is enforced at build time.** Production builds will fail on `any` mismatches, dead literal-type comparisons, etc., so don't paper over types you'll regret.

## Architecture

### Real, persisted backend for projects, assets, workflows, environments, endpoints, and runs
Everything is **persisted to disk** through API routes under `lib/server/`. Workflow execution is wired through a real DAG runtime that calls external LLM and ComfyUI providers. The FS under `.studio-data/` is the **single source of truth** — there is no mock-seed fallback in the services. A fresh install starts empty and the user creates entities through the UI.

`lib/nodeTemplates.ts` holds two static catalogs (not mock data, not seeded): the node palette (`nodeTemplates` — drives the workflow editor sidebar + drop defaults) and `imageComposerControls` (project composer right-side controls).

When adding features, **always call the API** — the only thing still mocked is the system-resource gauges in the sidebar StatusCards.

### Server-only services + folder layout

```
lib/server/
  storage.ts                      # path resolution, ID validation, ensureDir/readJson/writeJson
  projectService.ts               # projects CRUD + mock seeding
  assetService.ts                 # listAssets, getAsset(Sidecar|Binary), createAsset, deleteAsset, copyAsset
  workflowService.ts              # Studio workflow CRUD
  environmentService.ts           # ComfyUI environment CRUD + single-active invariant
  endpointService.ts              # Registered ComfyUI endpoints (per-environment)
  apiHelpers.ts                   # jsonError, errorToResponse — translate domain errors to HTTP
  workflow/
    runtime.ts                    # orchestrator (topo sort, executor harness, cancellation)
    runStore.ts                   # runs in memory + run.json/events.ndjson on disk
    eventBus.ts                   # per-runId pub/sub
    queue.ts                      # per-project FIFO queue
    introspect.ts                 # parse workflow_api.json → suggested input/output mappings
    executors/
      textInput | imageInput | workflowVariable
      textCombine | condition
      llm | comfyui | saveOutput
  providers/
    llm/
      base.ts | openai-compat.ts | anthropic.ts
    comfyui/
      client.ts                   # HTTP + WS adapter, auth dispatch, /upload/image, /view, /interrupt
```

Each module starts with `import "server-only";` so a stray client import becomes a build error. Never import these from a Client Component or shared `lib/` module that is also imported by Client Components.

Storage root is `STUDIO_DATA_DIR` (defaults to `<repo>/.studio-data`, gitignored). Layout:

```
.studio-data/
  blobs/                           # content-addressable asset binaries (dedup pool)
    <sha256>.<ext>                 # binary, one per unique content
    <sha256>.json                  # { hash, ext, size, createdAt, refCount }
  projects/<projectId>/
    project.json
    assets/<assetId>.json          # sidecar with `contentHash` pointing into blobs/
                                   # legacy assets keep <assetId>.<ext> binary in this dir

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lalantony/comfyui-ai-studio](https://github.com/lalantony/comfyui-ai-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
