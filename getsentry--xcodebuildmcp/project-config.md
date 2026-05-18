---
trigger: always_on
description: - `npm run build` - Build (wireit + tsup, ESM)
---

# Development Rules

## Build & Test
- `npm run build` - Build (wireit + tsup, ESM)
- `npm run test` - Unit/integration tests (Vitest)
- `npm run test:smoke` - Smoke tests (builds first, serial execution)
- `npm run lint` / `npm run lint:fix` - ESLint
- `npm run format` / `npm run format:check` - Prettier
- `npm run typecheck` - TypeScript type checking (src + test config)

## Architecture
ESM TypeScript project (`type: module`). Key layers:

- `src/cli/` - CLI entrypoint, yargs wiring, daemon routing
- `src/server/` - MCP stdio server, lifecycle, workflow/resource registration
- `src/runtime/` - Config bootstrap, session state, tool catalog assembly
- `src/core/manifest/` - YAML manifest loading, validation, tool module imports
- `src/mcp/tools/` - Tool implementations grouped by workflow (mirrors `manifests/workflows/`)
- `src/mcp/resources/` - MCP resource implementations
- `src/integrations/` - External integrations (Xcode tools bridge)
- `src/utils/` - Shared helpers (execution, logging, validation, responses)
- `src/visibility/` - Tool/workflow exposure predicates
- `src/daemon/` - Background daemon for persistent sessions
- `src/rendering/` - Output rendering and formatting
- `src/types/` - Shared type definitions

## Contributing Workflow
1. Create a branch from `main`
2. Make changes following the conventions in this file
3. Run the pre-commit checklist before committing:
   ```bash
   npm run lint:fix
   npm run typecheck
   npm run format
   npm run build
   npm test
   ```
4. Update `CHANGELOG.md` under `## [Unreleased]`
5. Update documentation if adding or modifying features
6. Clone and test against example projects (e.g., `XcodeBuildMCP-iOS-Template`) when changes affect runtime behavior
7. Push and create a pull request with a clear description
8. Link any related issues

## Code Quality
- No `any` types unless absolutely necessary
- Check node_modules for external API type definitions instead of guessing
- **NEVER use inline imports** - no `await import("./foo.js")`, no `import("pkg").Type` in type positions, no dynamic imports for types. Always use standard top-level imports.
- NEVER remove or downgrade code to fix type errors from outdated dependencies; upgrade the dependency instead
- Always ask before removing functionality or code that appears to be intentional
- Do not add fallback behavior by default. If required context, configuration, runtime state, or dependencies are missing, fail loudly and fix the caller/setup instead of silently switching to an alternate path. Add a fallback only when explicitly requested or when it is a documented product requirement.
- Follow TypeScript best practices

## Import Conventions
- ESM with explicit `.ts` extensions in `src/` (tsup rewrites to `.js` at build)
- No `.js` imports in `src/` (enforced by ESLint)
- No barrel imports from `utils/index` - import from specific submodules (e.g., `src/utils/execution/index.ts`, `src/utils/logging/index.ts`)


## Rendering and Streaming Contract
- Streaming fragments are transient live-progress output only. They may be displayed while a tool is running, but MUST NOT provide final settled MCP/JSON/CLI text.
- Final settled output MUST render from the final structured/domain result and next-step metadata. If final output needs data, add it to the final result type instead of reading it from fragments.
- Streaming-capable renderers may observe fragment callbacks only for live progress. Fragment handling must not affect final structured output or final settled text.

## Test Conventions
- Vitest with colocated `__tests__/` directories using `*.test.ts`
- Smoke tests in `src/smoke-tests/__tests__/` (separate Vitest config, serial execution)
- Use `vi.mock`/`vi.hoisted` for isolation; inject executors and mock file systems
- MCP integration tests use `McpServer`, `InMemoryTransport`, and `Client`
- External dependencies (command execution, file system) must use dependency injection via `createMockExecutor()` / `createMockFileSystemExecutor()` from `src/test-utils/`

## Tool Development
- Tool manifests in `manifests/tools/*.yaml` define `id`, `module`, `names.mcp` (snake_case), optional `names.cli` (kebab-case), predicates, and annotations
- Workflow manifests in `manifests/workflows/*.yaml` group tools and define exposure rules
- Tool modules export a Zod `schema`, a pure `*Logic` function, and a `handler` built with `createTypedTool` or `createSessionAwareTool`
- Resource modules export a `handler` (and a pure `*Logic` function); `uri`, `name`, `description`, and `mimeType` are declared in `manifests/resources/*.yaml`

## Commands
- NEVER commit unless user asks

## GitHub
When reading issues:
- Always read all comments on the issue
-
## Tools
- GitHub CLI for issues/PRs
- CLI design note: do not rely on CLI session-default writes. CLI is intentionally deterministic for CI/scripting and should use explicit command arguments as the primary input surface.
- When working on skill sources in `skills/`, use the `skill-creator` skill workflow.
- After modifying any skill source, run `npx skill-check <skill-directory>` and address all errors/warnings before handoff.
-
## Multi-process filesystem state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getsentry/XcodeBuildMCP](https://github.com/getsentry/XcodeBuildMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
