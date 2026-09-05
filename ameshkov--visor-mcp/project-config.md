---
trigger: always_on
description: Visor MCP is an MCP server that provides vision capabilities to
---

# AGENTS.md

Visor MCP is an MCP server that provides vision capabilities to
text-only models through OpenAI-compatible providers. It exposes seven
tools for image analysis, all fully implemented: `analyze_image`,
`ui_to_artifact`, `extract_text_from_screenshot`,
`diagnose_error_screenshot`, `understand_technical_diagram`,
`analyze_data_visualization`, and `ui_diff_check`.

## Table of Contents

- [Project Overview](#project-overview)
- [Technical Context](#technical-context)
- [Project Structure](#project-structure)
- [Build and Test Commands](#build-and-test-commands)
- [Contribution Instructions](#contribution-instructions)
- [Code Guidelines](#code-guidelines)
    - [Architecture](#architecture)
    - [Code Quality](#code-quality)
    - [Testing](#testing)
    - [Dependency Management](#dependency-management)
    - [Configuration & Documentation](#configuration--documentation)
    - [Markdown Formatting](#markdown-formatting)

## Project Overview

Visor MCP is a Model Context Protocol server that adds vision
capabilities to text-only LLMs by forwarding image-analysis requests to
an OpenAI-compatible Chat Completions endpoint. Instead of every model
needing native vision support, this server acts as a proxy: it accepts
an image source (data URL, HTTP/HTTPS URL, or absolute file path) and a
prompt, sends both to the provider, and returns the text response.

## Technical Context

| Field | Value |
| --- | --- |
| Language | TypeScript 5.9, ES2022 target, strict mode |
| Runtime | Node.js 24+ |
| Package Manager | pnpm 10+ |
| Framework | MCP SDK (`@modelcontextprotocol/sdk`) |
| Linting | ESLint 9.x + typescript-eslint + Knip |
| Formatting | Prettier 3.x, Markdownlint (markdownlint-cli2) |
| Project Type | MCP server (stdio transport) |

## Project Structure

```text
visor-mcp/
├── src/
│   ├── index.ts          # Entry point: load config, create server, start stdio
│   ├── config/           # Config loading + error formatting
│   ├── server/           # MCP server creation + tool handlers
│   │   └── tools/        # One file per tool + shared handler code
│   ├── services/         # Provider + images — each its own directory
│   │   ├── images/       # Image loading and validation
│   │   │   └── http-image.ts # HTTP image download path with retry + timeout
│   │   └── provider/     # OpenAI-compatible provider API integration
│   ├── utils/            # Shared helpers consumed across layers
│   │   ├── retry.ts      # Retry + per-attempt timeout driver
│   │   └── index.ts      # Barrel exports for the utils module
│   └── test/             # Test infrastructure colocated with source
│       ├── docs/          # Documentation-check tests
│       │   └── readme-topics.test.ts  # Asserts README covers all required operator topics
│       ├── e2e/          # End-to-end tests over stdio
│       ├── utils/        # Shared test helpers (mock servers, fixtures)
│       │   ├── image-fixtures.ts  # Tiny PNG/JPEG/WebP/GIF bytes + baseEnv
│       │   ├── mock-image-server.ts # MockImageServer + sequence/delay/hang
│       │   ├── mock-provider.ts  # MockProvider + sequence/delay/hang
│       │   ├── stdio-rpc.ts   # spawnServer + lineReader + request + init
│       │   └── temp-files.ts   # createTempDir + writeTempFile
│       └── setup.ts      # Shared test setup run by Vitest
├── fixtures/mcp-tester/  # Standalone E2E fixture runner (own pnpm project)
├── .github/workflows/ci.yml   # Quality gate + build
├── .husky/pre-commit          # Pre-commit hook running quality gate
├── eslint.config.mjs          # ESLint flat configuration
├── knip.config.ts             # Knip unused-export analysis configuration
├── package.json               # Project dependencies and scripts
├── tsconfig.json              # TypeScript configuration (production)
├── tsconfig.test.json         # TypeScript configuration (tests, noEmit)
└── vitest.config.ts           # Vitest configuration
```

Each `src/` subdirectory groups related modules behind a barrel
`index.ts` that defines its public API. Each service under `services/`
encapsulates all its functionality in its own directory with a barrel;
the top-level `services/index.ts` aggregates them into one public API.
The `utils/` module holds cross-cutting helpers (e.g., the retry driver)
shared across services. Unit tests are colocated with their source
(e.g., `src/services/images/images.test.ts` next to
`src/services/images/images.ts`). Shared test infrastructure and
end-to-end tests live under `src/test/`.

## Build and Test Commands

- `pnpm install` — install dependencies from the lockfile
- `pnpm build` — compile TypeScript to `build/` and make executable
- `pnpm typecheck` — check for TypeScript type errors in production and
  test code
- `pnpm lint` — lint source files with ESLint and check for unused
  exports with Knip
- `pnpm lint:fix` — lint and auto-fix issues
- `pnpm knip` — run Knip unused-export analysis separately
- `pnpm format:check` — check formatting with Prettier and Markdownlint
- `pnpm format:fix` — fix formatting issues
- `pnpm test` — run Vitest tests
- `pnpm test:watch` — run Vitest in watch mode
- `pnpm check` — run `format:check`, `lint`, `typecheck`, and `test`
  (full CI gate)
- `pnpm clean` — remove `node_modules` and `build/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ameshkov/visor-mcp](https://github.com/ameshkov/visor-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
