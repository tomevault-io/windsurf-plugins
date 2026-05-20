---
trigger: always_on
description: pnpm nx lint <project>                    # Lint (oxlint then eslint)
---

# AGENTS.md

## Commands

```bash
pnpm nx lint <project>                    # Lint (oxlint then eslint)
pnpm nx lint <project> --files=<path>     # Lint specific file(s) or glob
pnpm nx test <project> --watch=false      # Test
pnpm nx typecheck <project>              # Typecheck
pnpm nx build <project>                  # Build
pnpm nx serve ui                         # Web UI: runs production server after build (`NODE_ENV=production`)
pnpm nx serve example-electron           # Electron PoC: builds then launches packaged app (`NODE_ENV=production`)

pnpm infra:up / infra:down / infra:reset  # PostgreSQL + Redis (Docker)
pnpm db:generate                          # Generate Drizzle migrations
pnpm db:migrate                           # Run migrations
pnpm ci:affected                          # CI: affected tests, builds, lint, typecheck
pnpm docs:validate                        # Validate policy/research doc frontmatter
```

## Architecture

Tau is the AI-native CAD platform for the web (`tau.new`), built as an Nx monorepo with pnpm workspaces.

- **Frontend**: React Router v7, React 19, TypeScript, Tailwind CSS, Fumadocs
- **Backend**: NestJS API with Fastify, PostgreSQL (Drizzle ORM), Redis, Better Auth
- **CAD Engine**: Multi-kernel runtime (Replicad, JSCAD, Manifold, OpenSCAD, KCL)
- **AI**: LangGraph agent with tool-use (OpenAI, Anthropic, Vertex AI, Ollama, Together AI, Cerebras)

### Project Map

| Path                    | Description                                                                                                               |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `apps/ui`               | React Router v7 web app (CAD editor, file manager, AI chat, docs)                                                         |
| `apps/api`              | NestJS API (auth, database, chat WebSocket, LangGraph agent)                                                              |
| `packages/runtime`      | Multi-kernel CAD runtime — consumed as source via package.json exports                                                    |
| `kernels/openscad`      | `@taucad/openscad` — standalone GPL-2.0-or-later OpenSCAD kernel (isolates `openscad-wasm-prebuilt` from MIT runtime)     |
| `packages/react`        | React hooks for `@taucad/runtime` (useRender, useGeometryExport)                                                          |
| `packages/cli`          | `@taucad/cli` — headless CAD CLI (`taucad export <file> --ext=glb`)                                                       |
| `packages/converter`    | CAD file conversion (STL, STEP, IGES, DXF, glTF, USDZ)                                                                    |
| `packages/testing`      | Geometry analysis, grading, and test utilities (`analyzeGlb`, `evaluateRequirement`)                                      |
| `packages/memory`       | Shared-memory primitives (`SharedPool`, `SharedMemoryArena`)                                                              |
| `packages/telemetry`    | OTEL metric definitions, ingest schemas, observability runtime middleware                                                 |
| `packages/json-schema`  | JSON to JSON Schema inference                                                                                             |
| `packages/fs-client`    | UI file-manager facades (`FileContentService`, `FileTreeService`, `WorkerChangeChannel`); depends on `@taucad/filesystem` |
| `libs/chat`             | AI chat tool schemas, message schemas, RPC definitions                                                                    |
| `libs/types`            | Shared TypeScript types (API, project, CAD, file, graphics)                                                               |
| `libs/utils`            | Shared utilities (ID generation, path, file, schema, dispose)                                                             |
| `libs/units`            | Units of measurement and conversions                                                                                      |
| `libs/lsp-fs`           | LSP/workspace FS protocol (`@taucad/lsp-fs/protocol`, `sync` SAB channel)                                                 |
| `libs/lsp`              | Monaco LSP wiring: JSON-RPC bridge, URI workspace, TS worker entry, `lsp-fs` sync host                                    |
| `apps/ui/content/docs/` | Docs site (Fumadocs): `runtime/` and `editor/` sections                                                                   |

## Skills

Project skills in `.cursor/skills/` provide guided workflows. Read the relevant `SKILL.md` when performing these tasks:

| Skill                   | When to use                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------------ |
| `create-policy`         | Writing or updating `docs/policy/*.md` documents                                           |
| `create-research`       | Writing or updating `docs/research/*.md` investigation documents                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taucad/tau](https://github.com/taucad/tau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
