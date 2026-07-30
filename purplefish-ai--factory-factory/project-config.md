---
trigger: always_on
description: **Analysis Date:** 2026-05-17
---

# Coding Conventions

**Analysis Date:** 2026-05-17

## Naming Patterns

**Files:**
- Use lowercase kebab-case for feature modules, services, routers, hooks, helpers, and tests: `src/backend/services/workspace/service/state/kanban-state.ts`, `src/backend/trpc/project.trpc.ts`, `src/client/routes/projects/workspaces/use-workspace-detail.ts`, `src/lib/session-provider-selection.ts`.
- Use suffixes that identify the module role:
  - Backend tRPC routers: `*.trpc.ts`, e.g. `src/backend/trpc/workspace/init.trpc.ts`.
  - Backend tests: `*.test.ts`, `*.integration.test.ts`, or `*.manual.integration.test.ts`, e.g. `src/backend/routers/websocket/websocket.integration.test.ts`.
  - React component tests: `*.test.tsx`, e.g. `src/components/agent-activity/tool-renderers/tool-result-renderer.test.tsx`.
  - Storybook stories: `*.stories.tsx`, e.g. `src/components/ui/button.stories.tsx`.
- Service capsules use a fixed directory contract: `src/backend/services/{name}/index.ts`, `src/backend/services/{name}/service/`, and `src/backend/services/{name}/resources/`. New service files must follow this layout and export their public API from `src/backend/services/{name}/index.ts`.
- Shared cross-runtime contracts live under `src/shared/`, with explicit domain folders such as `src/shared/core/`, `src/shared/schemas/`, `src/shared/acp-protocol/`, and `src/shared/websocket/`.

**Functions:**
- Use `camelCase` for functions and methods: `computeKanbanColumn` in `src/backend/services/workspace/service/state/kanban-state.ts`, `sanitizeIssueTrackerConfig` in `src/shared/schemas/issue-tracker-config.schema.ts`, `createTrpcClient` in `src/client/lib/trpc.ts`.
- Use `create*` for factories and callers: `createContext` in `src/backend/trpc/trpc.ts`, `createWebSocketTestServer` in `src/backend/testing/websocket-test-utils.ts`, `createIntegrationDatabase` in `src/backend/testing/integration-db.ts`.
- Use `derive*` for pure state derivation helpers: `deriveWorkspaceRuntimeState` in `src/backend/services/workspace/service/state/workspace-runtime-state.ts`, `deriveWorkspaceSidebarStatus` in `src/shared/core/workspace-sidebar-status.ts`.
- Use `get*`, `find*`, `list*`, `update*`, and `delete*` verbs for resource and service methods. Keep data access verbs inside resource accessors such as `src/backend/services/workspace/resources/workspace.accessor.ts`.

**Variables:**
- Use `camelCase` for local variables, object fields, and parameters: `workspaceId`, `projectId`, `cachedKanbanColumn`, `stateComputedAt`.
- Use `PascalCase` for React components and classes: `KanbanCard` in `src/client/components/kanban/kanban-card.tsx`, `WorkspaceAccessor` in `src/backend/services/workspace/resources/workspace.accessor.ts`.
- Use `UPPER_SNAKE_CASE` for true constants and environment flag constants: `STALE_PROVISIONING_THRESHOLD_MS` in `src/backend/services/workspace/resources/workspace.accessor.ts`, `RUN_REAL_CODEX_APP_SERVER_TESTS` in `src/backend/services/session/service/acp/codex-app-server-adapter/codex-app-server-acp-adapter.manual.integration.test.ts`.
- Use explicit mock names in tests: `mockProjectManagementService` in `src/backend/trpc/project.router.test.ts`, `mockFindById` in `src/backend/services/workspace/service/state/kanban-state.test.ts`.

**Types:**
- Use `PascalCase` for interfaces, type aliases, enum-like imports, schemas, and services: `KanbanStateInput`, `WorkspaceWithKanbanState`, `UseSessionManagementReturn`, `IssueTrackerConfigSchema`.
- Export domain input/output types that callers need; keep implementation-only types local to the file. Examples: `WorkspaceWithSessions` is exported from `src/backend/services/workspace/resources/workspace.accessor.ts`, while `FindByProjectIdFilters` stays local.
- Prefer `interface` for object shapes consumed by implementations and `type` for derived Prisma/Zod/helper types: `UseSessionManagementOptions` in `src/client/routes/projects/workspaces/use-workspace-detail.ts`, `ConfigEnv = ReturnType<typeof ConfigEnvSchema.parse>` in `src/backend/services/config.service.ts`.
- Derive validation-backed types from Zod schemas with `z.infer`, as in `src/shared/schemas/issue-tracker-config.schema.ts` and `src/backend/schemas/tool-inputs.schema.ts`.

## Code Style

**Formatting:**
- Use Biome via `pnpm check:fix`; config is `biome.json`.
- Use 2-space indentation, 100-character line width, single quotes, semicolons, trailing commas where valid in ES5, and organized imports.
- Keep JSX self-closing when possible and use fragments where a wrapper element is unnecessary; Biome enforces `useSelfClosingElements` and `useFragmentSyntax` in `biome.json`.
- Use numeric separators for large numeric constants: `10_000` in `src/client/routes/projects/workspaces/use-workspace-detail.ts`, `30_000` in `src/backend/services/env-schemas.ts`.

**Linting:**
- Run `pnpm check` for Biome, environment-access checks, ownership checks, dependency-boundary checks, and Codex schema drift; run `pnpm check:fix` for formatting and fixable lints. Scripts are declared in `package.json`.
- Keep TypeScript strict and no-implicit-any compliant; `tsconfig.json` enables `strict`, `noImplicitAny`, `strictNullChecks`, `noUncheckedIndexedAccess`, and `noImplicitReturns`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [purplefish-ai/factory-factory](https://github.com/purplefish-ai/factory-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
