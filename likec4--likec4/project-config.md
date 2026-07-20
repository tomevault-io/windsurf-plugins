---
trigger: always_on
description: AGENTS.md is the canonical shared repository instruction file. Tool-specific files must import, point to, or defer to this file instead of duplicating repository guidance.
---

# Repository Guidelines

AGENTS.md is the canonical shared repository instruction file. Tool-specific files must import, point to, or defer to this file instead of duplicating repository guidance.

LikeC4 is an architecture-as-code tool for visualizing software architecture. It provides a DSL for describing architecture, a language server, CLI, VSCode extension, and web-based diagram visualization.

## Project Structure & Module Organization

- Monorepo managed by `pnpm` workspaces and `turbo`.
- `apps/` contains user-facing apps, notably `apps/docs` and `apps/playground`.
- `packages/likec4/` is the CLI and static site generator main entry point.
- `packages/likec4-spa/` is the single-page application for rendering architecture diagrams.
- `packages/vite-plugin/` is the LikeC4 Vite plugin (`@likec4/vite-plugin`).
- `packages/lsp/` is the standalone LikeC4 Language Server for editor integrations (`@likec4/lsp`).
- `packages/react/` is the LikeC4 React bundle (`@likec4/react`).
- `packages/core/` contains core and model types, model builder, compute-view, and layout drift detection logic.
- `packages/language-server/` contains the Langium-based DSL parser and LSP implementation.
- `packages/language-services/` contains language service initialization for browser and Node.js consumers.
- `packages/diagram/` contains the React/ReactFlow diagram renderer.
- `packages/layouts/` contains Graphviz-based layout algorithms.
- `packages/generators/` contains exports to Mermaid, PlantUML, D2, and LikeC4 DSL.
- `packages/vscode/` contains the VSCode extension.
- `packages/vscode-preview/` contains the preview panel component for VSCode.
- `packages/config/` contains configuration schema and validation.
- `packages/icons/` is script-generated; never change it unless explicitly asked.
- `packages/log/` contains shared logging utilities.
- `packages/mcp/` contains the MCP server package; see `packages/mcp/README.md`.
- `packages/tsconfig/` contains shared TypeScript configuration.
- `packages/create-likec4/` is not used for now.
- `e2e/` is an isolated workspace for Playwright end-to-end tests.
- `styled-system/preset` holds the PandaCSS preset.
- `styled-system/styles` holds `pandacss codegen` results shared across packages.
- `examples/` provides sample LikeC4 projects.
- `devops/` contains CI/CD and repository utility scripts.
- `skills/` contains agent skills for AI assistants, including `skills/likec4-dsl/`.
- LeanIX / Draw.io bridge details live in `packages/leanix-bridge/README.md` and `skills/likec4-dsl/references/bridge-leanix-drawio.md`.

## App ↔ Language Server Architecture

The webapp talks to the Language Server through three layered packages. When extending the flow, pick the right layer:

- `packages/diagram` is the UI contract. It renders diagrams from a `LikeC4Model`, defines the consumer contract via `LikeC4ModelProvider` in `packages/diagram/src/LikeC4ModelProvider.tsx`, and exposes `useLikeC4Model`-style hooks. It must not know how data is fetched, stored, or mutated.
- `packages/likec4-spa` is the UI host. It materializes the diagram into a runnable app, imports `likec4:*` virtual modules, wraps them in nanostores for HMR reactivity, and mounts `LikeC4ModelProvider`. See `packages/likec4-spa/src/context/LikeC4ModelContext.tsx`.
- `packages/vite-plugin` is the data and RPC bridge. It owns the `likec4:*` virtual modules in `packages/vite-plugin/src/virtuals/` and the birpc channel between SPA and Language Server over Vite HMR in `packages/vite-plugin/src/rpc/rpc.ts` and `packages/vite-plugin/src/virtuals/rpc.ts`.

Dev-mode flow: `SPA → likec4:rpc (birpc over import.meta.hot) → vite-plugin (server.hot.on) → @likec4/language-services → updated likec4:model → HMR → SPA nanostore → diagram re-renders`.

In production builds (`likec4 build`), RPC is absent and virtual modules are inlined as static JSON. The diagram is read-only.

### Where to make changes

- New visual feature, no new data: change `packages/diagram` only.
- New data shape from the model: add a virtual module in `packages/vite-plugin`, surface a hook or atom in `packages/likec4-spa`, then consume it in `packages/diagram`.
- New action that mutates the model through a round trip to the LSP: add an RPC method in `packages/vite-plugin/src/rpc`, then call it from `packages/likec4-spa`.
- Do not import `@likec4/language-server` directly from `packages/likec4-spa`; always go through the vite-plugin RPC.

## Public API entry files

- `packages/likec4/src/LikeC4.ts` is the SDK entry for `import { LikeC4 } from 'likec4'`.
- `packages/likec4/src/index.ts` contains SDK re-exports, including `writeDSL`.
- `packages/language-services/src/common/LikeC4.ts` contains the real `LikeC4` class.
- `packages/language-services/src/node/index.ts` contains Node-only helpers such as `fromWorkspace`, `fromSource`, and `writeDSL`.
- `packages/core/src/builder/Builder.ts` contains `Builder` and static helpers `forSpecification`, `specification`, and `fromParsed`.
- `packages/core/src/builder/_types.ts` contains Builder type machinery: `Types`, `AnyTypes`, `Types.FromAux`, and `Types.ToAux`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [likec4/likec4](https://github.com/likec4/likec4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
