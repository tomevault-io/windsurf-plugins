---
trigger: always_on
description: This file is the onboarding note for AI coding agents working in ImageX. Keep it current, concise, and open-ended: update it when architecture, commands, storage, or product direction changes.
---

# AGENTS.md

This file is the onboarding note for AI coding agents working in ImageX. Keep it current, concise, and open-ended: update it when architecture, commands, storage, or product direction changes.

## Project Purpose

ImageX is a local-first, node-based image generation workflow editor. Users compose prompts, image references, colors, files, image-processing steps, and AI output nodes into a DAG; the app previews image edits locally and compiles the graph into structured JSON prompts plus image references for generation.

The current checkout is a Vite/React web app plus a local Express daemon. Do not assume an Electron wrapper exists unless the codebase gains one.

## Current Shape

- Package manager: `npm` with `package-lock.json`; do not switch package managers casually.
- Runtime: Node.js 20+, ESM TypeScript, React 19, Vite, Tailwind CSS v4, shadcn/ui, React Flow, Zustand, raw WebGL frontend image pipelines, photon-node daemon transforms, Express.
- Main commands:
  - Install: `npm install`
  - Dev app: `npm run dev` starts daemon on `127.0.0.1:3847` and Vite on `127.0.0.1:5173`
  - Web only: `npm run dev:web`
  - Daemon only: `npm run dev:daemon`
  - Remote bind, only on a trusted network: `npx tsx src/cli/index.ts ui --host 0.0.0.0 --allow-remote`
  - Type check: `npm run check`
  - Tests: `npm test` runs typecheck, unit tests, WebGL browser verification, and workflow E2E verification
  - WebGL verifier only: `npm run test:webgl`
  - Workflow E2E verifier only: `npm run test:e2e`
  - Production build: `npm run build`
  - Start built daemon/UI: `npm start`
- CLI commands:
  - First-run app: `imagex` checks Codex auth, prompts login when missing/stale, starts the local daemon on `127.0.0.1:3847`, and opens the built web UI.
  - Auth: `imagex auth` or `npx tsx src/cli/index.ts auth`
  - Status: `imagex status`, `imagex doctor`, or the matching `npx tsx src/cli/index.ts ...` source commands. `imagex whoami` remains a compatibility alias for `status`.
  - Logout: `imagex logout` or `npx tsx src/cli/index.ts logout`
  - Automation/dev server: `imagex ui --no-open` or `npx tsx src/cli/index.ts ui --no-open` starts the daemon/UI without the first-run auth prompt.

## Read These First

- `README.md` for the human-facing overview and quickstart.
- `package.json`, `vite.config.ts`, `tsconfig.json`, and `components.json` for tooling.
- `src/shared/types.ts` for persisted workflow, project, node, asset, and generation schemas.
- `src/web/ui/flow/meta.ts`, `src/web/ui/flow/ports.ts`, and `src/web/ui/flow/fields/definitions.ts` for node catalog, port compatibility, and field definitions.
- `src/workflows/compiler.ts` for graph-to-prompt compilation.
- `src/daemon/server.ts` for API routes, durable generation jobs, output-node run planning, image-reference resolution, and server-side image transforms.
- `src/providers/codexImage.ts` for the Codex Responses image tool transport. `CODEX_API_BASE` can point to a local mock endpoint during development.
- `src/projects/store.ts` and `src/workflows/store.ts` for local persistence.
- `src/web/state/flowStore.ts`, `src/web/state/graphEngine.ts`, and `src/web/ui/flow/imaging/` for the performance-sensitive editor and preview pipeline.
- `src/web/ui/App.tsx`, `src/web/styles.css`, `src/web/ui/editor/TopBar/`, `src/web/ui/editor/Sidebar/`, `src/web/ui/editor/SidePanel/`, and `src/web/ui/editor/InspectorPanel/` for the floating editor shell.
- `src/web/ui/editor/useEditorActions.ts` and `src/web/ui/editor/useProjectActions.ts` for workflow, project, generation, asset, undo/redo, and autosave behavior.

Gitignored local notes may exist. Treat them as private context only; do not quote unpublished details into tracked files.

## Architecture Invariants

- Persisted workflow data uses `ImageXWorkflow`, `ImageXNode`, and `ImageXEdge` from `src/shared/types.ts`. React Flow nodes are adapters around those workflow nodes, not a separate source of truth.
- `FlowStore` owns React Flow nodes and edges. `FlowEditor` reads them through store hooks, keeps React Flow props stable, and lets React Flow handle transient drag positions. Durable workflow sync happens on drag stop; selection, dimension, and position-only changes must not wake `GraphEngine`.
- Keep React Flow handle dragging as the only connection gesture. `FlowEditor` sets `connectOnClick={false}` because click-to-connect can leave handles in a bad pointer-event state after interrupted connection attempts.
- Dynamic handles must refresh React Flow internals when their ids or positions change. `BaseNode` owns this for node-level input/output handle signatures, including output image sockets that appear after generation.
- ImageX node components use a custom `React.memo` comparator that ignores React Flow position and dragging props. The wrapper owns live movement; node content should re-render only for rendered data, selection, type/id, or connectability changes.
- Keep React Flow handles visible outside node borders. Do not use paint containment on `.react-flow__node` or `.ix-node`; scope containment to internal preview/media boxes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xikhar/imagex](https://github.com/xikhar/imagex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
