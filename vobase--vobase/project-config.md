---
trigger: always_on
description: Full-stack TypeScript framework for AI coding agents. Bun + Hono + Drizzle + PostgreSQL, with a first-class agent runtime built on `pi-agent-core` + `pi-ai`.
---

# Vobase Monorepo

Full-stack TypeScript framework for AI coding agents. Bun + Hono + Drizzle + PostgreSQL, with a first-class agent runtime built on `pi-agent-core` + `pi-ai`.

## Packages

- `@vobase/core` — runtime engine: `ModuleDef` contract, agent harness (frozen-snapshot wakes, steer/abort, tool budget spill, idle resumption, restart recovery), workspace primitives (materializers, RO enforcer, AGENTS.md generator, CLI verb registry), realtime (LISTEN/NOTIFY + SSE), HMAC + webhooks, jobs (pg-boss), HTTP client, audit/sequences/auth/storage/channels/integrations schemas + adapters.
- `@vobase/template` — agent-native helpdesk scaffold (private). Canonical module shape, `wake/` harness, `pi-agent-core` agent runtime. See `packages/template/CLAUDE.md` for conventions.
- `@vobase/cli` — standalone, catalog-driven CLI binary. Discovers tenant verbs at runtime via `/api/cli/verbs`; the same binary works across deployments with different module sets.
- `create-vobase` — project scaffolder via `bun create vobase`.
- `legacy/template-v1` — frozen pre-canonical template (Mastra + declarative config). Out of the workspace, pinned to `@vobase/core@0.33.0`. Do not modify.

## Commands

`bun install` | `bun run dev` | `bun run build` | `bun run test` | `bun run lint` | `bun run typecheck`

A whitelist-mode pre-commit hook gates `bun.lock` against the `workspaces` glob in root `package.json` to prevent macOS Bun from auto-discovering nested package.json files (e.g. `poc/`) and breaking Linux CI's frozen install. Enable once per clone: `git config core.hooksPath .githooks`. Manual run: `bun run check:lockfile`.

## Quality Rules

Every change must be clean, high quality, and maintainable. No exceptions.

- End-to-end type safety: Drizzle for queries, Zod validation on all inputs, Hono typed RPC client, TanStack generated routes, TanStack Query (not raw fetch).
- No `any`, no `as` casts unless provably safe, no `// @ts-ignore`.
- Every handler validates input with Zod. Return typed errors via `VobaseError` factories.
- Tests for every feature. Colocate tests next to implementation (`*.test.ts`).
- Zero tech debt tolerance: fix what you touch. No TODO comments without linked issues.
- Biome for formatting + linting. Run `bun run lint` before committing.
- Dynamic `import()` only for: heavy optional deps (MCP SDK, AI SDK, googleapis, mammoth, etc.), config-gated features, test mocking. Local module imports must be static.
- Prefer Bun native APIs over `node:*` modules: `Bun.file()`, `Bun.write()`, `Bun.spawnSync()`, `Bun.Glob`, `$` shell. Use `node:path` and `node:fs` only when no Bun equivalent exists.
- Search existing patterns before writing new code. Mirror established naming and error handling.

## Architecture

### Core surface (`@vobase/core`)

Core ships **primitives**, not a fixed list of `_`-prefixed modules. Public API by area:

- **Module contract** — `ModuleDef`, `ModuleInitCtx`, `bootModules`, `sortModules`, `RoHintFn`, `AgentContributions`, `collectAgentContributions`, `collectJobs`, `collectWebRoutes`.
- **Agent harness** — `createHarness`, `HarnessHandle`, `HarnessEvent` union, `defineAgentTool`, `AgentTool`, `WakeRuntime`, `WakeScope`, frozen-snapshot tooling (`buildFrozenSnapshot`, `assertFrozenForWake`), steer queue, journal service, journaled tx, dispatch + concurrency gate, cost cap, idle resumption, restart recovery, subagent registry, turn budget, side-load collector, message history, `llmCall`.
- **Workspace** — `createWorkspace`, `MaterializerRegistry`, `WorkspaceMaterializer`, `WorkspaceMaterializerFactory`, `IndexFileBuilder`, `IndexContributor`, `defineIndexContributor`, `generateAgentsMd`, `ScopedFs`, `ReadOnlyConfig`, `buildReadOnlyConfig`, `DirtyTracker`, `snapshotFs`.
- **CLI registry** — `CliVerbRegistry`, `defineCliVerb`, `createCatalogRoute`, `createCliDispatchRoute`, `createInProcessTransport`, `createBashVobaseCommand`, `parseBashArgv`, `coerceBashArgs`, `renderBashHelp`, `renderBashResult`.
- **Adapters + contracts** — `AuthAdapter`, `ChannelAdapter`, `StorageAdapter`, `Permission`, `OrganizationContext`, `ChangePayload`. Adapter creators: `createResendAdapter`, `createSmtpAdapter`, `createWhatsAppAdapter`, `createLocalAdapter`, `createS3Adapter`.
- **Schemas** (Drizzle) — `auditLog`, `recordAudits`, `authUser`/`authSession`/`authOrganization`/etc., `channelsLog`, `channelsTemplates`, `sequences`, `storageObjects`, `webhookDedup`, `integrationsTable`, `auditPgSchema`/`authPgSchema`/`harnessPgSchema`/`infraPgSchema`, harness schemas (`activeWakes`, `agentMessages`, `conversationEvents`, `pendingApprovals`, `tenantCostDaily`, `threads`, `auditWakeMap`).
- **Declarative resources** — `defineDeclarativeResource`, `bindDeclarativeTable`, `parseFileBytes`, `serializeMarkdownFrontmatter`, `serializeYaml`.
- **HMAC + webhooks** — `signHmac`, `verifyHmacSignature`, `createWebhookRoutes`, `webhookDedup`.
- **Realtime** — `createRealtimeService`, `createNoopRealtime`, `RealtimeService`.
- **Jobs** — `defineJob`, `createWorker`, `createScheduler`, `Scheduler`, `JobOptions`.
- **HTTP** — `createHttpClient`, `CircuitBreaker`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vobase/vobase](https://github.com/vobase/vobase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
