---
trigger: always_on
description: This is a **pnpm monorepo** with two MCP implementations:
---

# Copilot Instructions for Storybook MCP Addon

## Architecture Overview

This is a **pnpm monorepo** with two MCP implementations:

- **`packages/addon-mcp`**: Storybook addon using `tmcp`, exposes MCP server at `/mcp` via Vite middleware
  - Provides addon-specific tools (story URLs, UI building instructions)
  - **Imports and reuses tools from `@storybook/mcp` package** for component manifest features
  - Extends `StorybookContext` with addon-specific configuration (`AddonContext`)
- **`packages/mcp`**: Standalone MCP library using `tmcp`, reusable outside Storybook
  - Provides reusable component manifest tools (list components, get documentation)
  - Exports tools and types for consumption by addon-mcp
- **`apps/internal-storybook`**: Test environment for addon integration

**Both packages use `tmcp`** with HTTP transport and Valibot schema validation for consistent APIs.

### Addon Architecture

The addon uses a **Vite plugin workaround** to inject middleware (see `packages/addon-mcp/src/preset.ts`):

- Storybook doesn't expose an API for addons to register server middleware
- Solution: Inject a Vite plugin via `viteFinal` that adds `/mcp` endpoint
- Handler in `mcp-handler.ts` creates MCP servers using `tmcp` with HTTP transport

**Toolset Configuration:**

The addon supports configuring which toolsets are enabled:

- **Addon Options**: Configure default toolsets in `.storybook/main.js`:
  ```typescript
  {
    name: '@storybook/addon-mcp',
    options: {
      toolsets: {
        dev: true,
        docs: true,
      }
    }
  }
  ```
- **Per-Request Override**: MCP clients can override toolsets per-request using the `X-MCP-Toolsets` header:
  - Header format: comma-separated list (e.g., `dev,docs`)
  - When header is present, only specified toolsets are enabled (others are disabled)
  - When header is absent, addon options are used
- **Tool Enablement**: Tools use the `enabled` callback to check if their toolset is active:
  ```typescript
  server.tool(
  	{
  		name: 'my-tool',
  		enabled: () => server.ctx.custom?.toolsets?.dev ?? true,
  	},
  	handler,
  );
  ```
- **Context-Aware**: The `getToolsets()` function in `mcp-handler.ts` parses the header and returns enabled toolsets, which are passed to tools via `AddonContext.toolsets`

### MCP Library Architecture

The `@storybook/mcp` package (in `packages/mcp`) is framework-agnostic:

- Uses `tmcp` with HTTP transport and Valibot schema validation
- Factory pattern: `createStorybookMcpHandler()` returns a request handler
- Context-based: handlers accept `StorybookContext` which includes the HTTP `Request` object and optional callbacks
- **Exports tools and types** for reuse by `addon-mcp` and other consumers
- **Request-based manifest loading**: The `request` property in context is passed to tools, which use it to determine the manifest URL (defaults to same origin, replacing `/mcp` with the manifest path)
- **Optional manifestProvider**: Custom function to override default manifest fetching behavior
  - Signature: `(request: Request, path: string) => Promise<string>`
  - Receives the `Request` object and a `path` parameter (currently always `'./manifests/components.json'`)
  - The provider determines the base URL (e.g., mapping to S3 buckets) while the MCP server handles the path
  - Returns the manifest JSON as a string
- **Optional handlers**: `StorybookContext` supports optional handlers that are called at various points, allowing consumers to track usage or collect telemetry:
  - `onSessionInitialize`: Called when an MCP session is initialized
  - `onListAllDocumentation`: Called when the list-all-documentation tool is invoked
  - `onGetDocumentation`: Called when the get-documentation tool is invoked
- **Output Format**: Responses are markdown-only (token-efficient markdown with adaptive formatting).
  - Formatter implementations are in `packages/mcp/src/utils/manifest-formatter/`.

## Development Environment

**Prerequisites:**

- Node.js **24+** (enforced by `.nvmrc`)
- pnpm **10.19.0+** (strict `packageManager` in root `package.json`)

**Monorepo orchestration:**

- Turborepo manages build dependencies (see `turbo.json`)
- Run `pnpm dev` at root for parallel development
- Run `pnpm storybook` to test addon (starts internal-storybook + addon dev mode)

**Build tools:**

- All packages use `tsdown` (rolldown-based bundler)
- Shared configuration in `tsdown-shared.config.ts` at monorepo root
- Individual packages extend shared config in their `tsdown.config.ts`

**Testing:**

- **Unit tests**: Both `packages/mcp` and `packages/addon-mcp` have unit tests (Vitest with coverage)
  - Run `pnpm test run --coverage` in individual package directories
  - Run `pnpm test:run` at root to run all unit tests
  - Prefer TDD when adding new tools
- **E2E tests**: `apps/internal-storybook/tests` contains E2E tests for the addon
  - Run `pnpm test` in `apps/internal-storybook` directory
  - Tests verify MCP endpoint works with latest Storybook prereleases
  - Uses inline snapshots for response validation
  - **When to update E2E tests**:
    - Adding or modifying MCP tools (update tool discovery snapshots)
    - Changing MCP protocol implementation (update session init tests)
    - Modifying tool responses or schemas (update tool-specific tests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [storybookjs/mcp](https://github.com/storybookjs/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
