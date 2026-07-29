---
trigger: always_on
description: **Analysis Date:** 2026-05-12
---

# Coding Conventions

**Analysis Date:** 2026-05-12

## Naming Patterns

**Files:**
- Use `kebab-case` for TypeScript source files across apps and packages, e.g. `apps/gateway/src/services/session-router.ts`, `apps/webui/src/lib/chat-store.ts`, `apps/mobvibe-cli/src/config-loader.ts`, and `packages/shared/src/types/socket-events.ts`.
- React component implementation files may use `PascalCase.tsx` when the component is the file boundary, e.g. `apps/webui/src/components/app/ChatFooter.tsx` and `apps/webui/src/components/app/AppSidebar.tsx`.
- Store files use `*-store.ts`, e.g. `apps/webui/src/lib/chat-store.ts`, `apps/webui/src/lib/ui-store.ts`, and `apps/webui/src/lib/machines-store.ts`.
- Test files use `*.test.ts` / `*.test.tsx` for unit and component tests, and Playwright E2E specs use `*.spec.ts`, e.g. `apps/webui/tests/e2e/session-restore.spec.ts`.

**Functions:**
- Use `camelCase` for functions and helpers, e.g. `normalizeBackendIds` in `apps/webui/src/hooks/useSessionQueries.ts`, `requestJsonWithTimeout` in `apps/webui/src/lib/api.ts`, and `createMockSessionSummary` in `apps/gateway/src/services/__tests__/session-router.test.ts`.
- React hooks must use `useX` naming and live under `apps/webui/src/hooks/`, e.g. `useSessionQueries` and `useDiscoverSessionsMutation` in `apps/webui/src/hooks/useSessionQueries.ts`.
- Type guard helpers should use `isX`, e.g. `isPromptImageFile` in `apps/webui/src/components/app/ChatFooter.tsx` and `isAbortError` in `apps/webui/src/lib/api.ts`.
- Factory/test helpers should use `createX` or `buildX`, e.g. `createFallbackError` in `apps/webui/src/lib/error-utils.ts`, `buildRequestValidationError` in `apps/gateway/src/routes/sessions.ts`, and `createImageBlock` in `packages/shared/tests/prompt-images.test.ts`.

**Variables:**
- Use `camelCase` for locals and parameters, e.g. `backendCapabilities`, `lastError`, and `hasExplicitBackendSelection` in `apps/webui/src/hooks/useSessionQueries.ts`.
- Use `UPPER_SNAKE_CASE` for module-level constants with fixed configuration values, e.g. `SEND_MESSAGE_TIMEOUT_MS` and `SESSION_LOAD_TIMEOUT_MS` in `apps/webui/src/lib/api.ts`, and `RPC_TIMEOUT` in `apps/gateway/src/services/session-router.ts`.
- Prefix intentionally unused destructured props/parameters with `_`, e.g. `_size` in `apps/webui/src/components/app/__tests__/ChatFooter.test.tsx`.

**Types:**
- Use `PascalCase` for exported types, classes, and React props, e.g. `ChatSession`, `ChatMessage`, and `SessionListEntry` in `apps/webui/src/lib/chat-store.ts`, `ApiError` in `apps/webui/src/lib/api.ts`, and `SessionRouter` in `apps/gateway/src/services/session-router.ts`.
- Prefer explicit discriminated unions for state and messages, e.g. `ChatMessage` and message `kind` variants in `apps/webui/src/lib/chat-store.ts`.
- Use `unknown` for untrusted error/input values and narrow before use, e.g. `normalizeError(error: unknown, ...)` in `apps/webui/src/lib/error-utils.ts` and `getErrorMessage(error: unknown)` in `apps/gateway/src/routes/sessions.ts`.

## Code Style

**Formatting:**
- Tool: Biome 2.3.11 configured in `biome.json` and package overrides such as `packages/ui/biome.json`.
- Use tabs for indentation (`biome.json` line 14) and double quotes for JavaScript/TypeScript strings (`biome.json` line 24).
- Do not manually organize imports; Biome source action `organizeImports` is enabled in `biome.json` and `packages/ui/biome.json`.
- Keep TypeScript strict. `strict: true` is enabled in `apps/webui/tsconfig.app.json`, `apps/gateway/tsconfig.json`, `apps/mobvibe-cli/tsconfig.json`, and `packages/shared/tsconfig.json`.

**Linting:**
- Tool: Biome check. Root scripts are `pnpm format`, `pnpm lint`, `pnpm format:check`, and `pnpm lint:check` in `package.json`.
- Package scripts run `biome check --write .` and `biome format --write .`, e.g. `apps/webui/package.json`, `apps/gateway/package.json`, `apps/mobvibe-cli/package.json`, `packages/shared/package.json`, and `packages/ui/package.json`.
- `packages/ui/biome.json` keeps recommended rules but warns on accessibility and disables `suspicious` and `style`; UI package changes should still follow root conventions unless this override is intentional.

## Import Organization

**Order:**
1. Node built-ins and external packages, e.g. `node:crypto`, `@mobvibe/shared`, `socket.io`, `@tanstack/react-query` in `apps/gateway/src/services/session-router.ts` and `apps/webui/src/components/app/ChatFooter.tsx`.
2. Workspace packages such as `@mobvibe/shared` and `@mobvibe/ui/*`, e.g. `apps/webui/src/components/app/ChatFooter.tsx` and `apps/webui/src/pages/LegalPage.tsx`.
3. App aliases, e.g. `@/components/*`, `@/hooks/*`, and `@/lib/*` in `apps/webui/src/components/app/ChatFooter.tsx`.
4. Relative imports from the same package/module, e.g. `../services/session-router.js` in `apps/gateway/src/services/__tests__/session-router.test.ts` and `./error-utils` in `apps/webui/src/lib/api.ts`.

**Path Aliases:**
- WebUI uses `@/*` for `apps/webui/src/*`, configured in `apps/webui/tsconfig.json`, `apps/webui/tsconfig.app.json`, and `apps/webui/vitest.config.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eric-Song-Nop/mobvibe](https://github.com/Eric-Song-Nop/mobvibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
