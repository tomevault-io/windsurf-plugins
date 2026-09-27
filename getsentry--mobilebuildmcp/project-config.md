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
- Review the complete merge-base diff and trace changed or reused helper contracts, including error and sentinel returns, through callers, consumers, tests, and operational configuration.
- Verify standard quality commands include every changed path and exercise exact entry points and argument variants; validate explicitly when they do not.
- For asynchronous, workflow, or process-boundary changes, enumerate lifecycle states, retries, supersession, and race transitions; test terminal outcomes and missing or optional metadata.

## Import Conventions
- ESM with explicit `.ts` extensions in `src/` (tsup rewrites to `.js` at build)
- No `.js` imports in `src/` (enforced by ESLint)
- No barrel imports from `utils/index` - import from specific submodules (e.g., `src/utils/execution/index.ts`, `src/utils/logging/index.ts`)


## Rendering and Streaming Contract
- Streaming fragments are transient live-progress output only. They may be displayed while a tool is running, but MUST NOT provide final settled MCP/JSON/CLI text.
- Final settled output MUST render from the final structured/domain result and next-step metadata. If final output needs data, add it to the final result type instead of reading it from fragments.
- Streaming-capable renderers may observe fragment callbacks only for live progress. Fragment handling must not affect final structured output or final settled text.

## Error Handling
- Structured errors (domain results with `didError`) are for domain errors only: failures in the user's build/test/device/simulator workflow (compile errors, test failures, missing destinations, etc.).
- System errors with the MCP server or CLI itself (invalid internal state, unresolvable configuration, infrastructure failures) must NOT be wrapped in structured domain results — let them surface as runtime tool errors so they are clearly distinguishable from workflow outcomes.

## Test Conventions
- Vitest with colocated `__tests__/` directories using `*.test.ts`
- Snapshot tests (`*.snapshot.test.ts`) must only assert generated tool output against fixtures. Move helper, parser, schema, setup, or behavior assertions to non-snapshot unit/integration tests.
- Smoke tests in `src/smoke-tests/__tests__/` (separate Vitest config, serial execution)
- Use `vi.mock`/`vi.hoisted` for isolation; inject executors and mock file systems
- MCP integration tests use `McpServer`, `InMemoryTransport`, and `Client`
- External dependencies (command execution, file system) must use dependency injection via `createMockExecutor()` / `createMockFileSystemExecutor()` from `src/test-utils/`

## Tool Development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getsentry/MobileBuildMCP](https://github.com/getsentry/MobileBuildMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
