---
trigger: always_on
description: MCP Server for the Austrian Legal Information System (RIS - Rechtsinformationssystem).
---

# CLAUDE.md

MCP Server for the Austrian Legal Information System (RIS - Rechtsinformationssystem).

## Quick Start

```bash
pnpm install
pnpm run build
```

## Development Commands

```bash
pnpm run dev             # Start with tsx (hot reload, stdio)
pnpm run dev:http        # Start HTTP server with tsx (hot reload)
pnpm run build           # Compile TypeScript (runs typecheck first)
pnpm start               # Run compiled version (stdio)
pnpm run start:http      # Run HTTP server (Streamable HTTP transport)
pnpm run check           # Run typecheck + lint + format:check + tests
```

## Testing

```bash
pnpm test                # Run all unit tests (744 tests, 10 files)
pnpm run test:watch      # Run tests in watch mode
pnpm run test:coverage   # Tests with V8 coverage report
pnpm run test:integration # Integration tests (separate config, requires network)
```

### Manual Testing with MCP Inspector

```bash
pnpm run inspect
```

## Code Quality

```bash
pnpm run typecheck       # TypeScript strict mode check
pnpm run lint            # ESLint (strict + stylistic rules)
pnpm run lint:fix        # ESLint with auto-fix
pnpm run format          # Prettier format
pnpm run format:check    # Prettier check
```

Pre-commit hooks (Husky) auto-run `prettier --write` and `eslint --fix` on staged `.ts` files. Commits must follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `chore:`, etc.) — enforced by commitlint.

## Code Architecture

```
src/
├── index.ts           # Entry point (stdio transport)
├── http.ts            # Entry point (Streamable HTTP transport, Express)
├── server.ts          # MCP server init, delegates to tools/
├── client.ts          # HTTP client for RIS API, error classes, URL construction
├── types.ts           # Zod schemas + TypeScript types
├── parser.ts          # JSON parsing and response normalization
├── formatting.ts      # Output formatting (markdown/json), character truncation
├── helpers.ts         # Shared helper functions for tool handlers
├── constants.ts       # Static mappings, enum values, configuration
├── version.ts         # Shared VERSION constant (read from package.json)
├── tools/
│   ├── index.ts       # registerAllTools() barrel file
│   ├── bundesrecht.ts
│   ├── landesrecht.ts
│   ├── judikatur.ts
│   ├── bundesgesetzblatt.ts
│   ├── landesgesetzblatt.ts
│   ├── regierungsvorlagen.ts
│   ├── dokument.ts    # Full document retrieval (largest handler)
│   ├── bezirke.ts
│   ├── gemeinden.ts
│   ├── sonstige.ts    # 8 sub-applications (second largest)
│   ├── history.ts
│   └── verordnungen.ts
└── __tests__/
    ├── client.test.ts
    ├── document-matching.test.ts
    ├── http.test.ts
    ├── edge-cases.test.ts
    ├── formatting.test.ts
    ├── history.test.ts
    ├── parser.test.ts
    ├── security.e2e.test.ts
    ├── server.test.ts
    ├── types.test.ts
    └── integration/
        └── smoke.test.ts
```

## Key Patterns

### Adding/Modifying a Tool Handler

Each tool lives in `src/tools/<name>.ts` and exports a `register<Name>Tool(server)` function. Pattern:

1. Register with `server.registerTool(name, { title, description, inputSchema, annotations }, handler)` — `title` is a German display name, `description`/`inputSchema` are English, `annotations` is `{ readOnlyHint: true, openWorldHint: true }` for these read-only tools. (The deprecated `server.tool(...)` overload is no longer used.)
2. For `limit`/`seite`, reuse `LimitSchema`/`SeiteSchema` from `types.ts` instead of raw `z.number()`
3. Use `helpers.ts` functions: `hasAnyParam()`, `buildBaseParams()`, `addOptionalParams()`, `executeSearchTool()`
4. Call client search functions from `client.ts`
5. Register in `src/tools/index.ts` if adding a new tool

### Helper Functions (helpers.ts)

| Function | Purpose |
|----------|---------|
| `createMcpResponse()` | Standard MCP text response |
| `createValidationErrorResponse()` | Validation error listing required params |
| `hasAnyParam()` | Check if any specified param has a truthy value |
| `buildBaseParams()` | Build base API params (Applikation, DokumenteProSeite, Seitennummer) |
| `addOptionalParams()` | Add truthy optional params to request |
| `executeSearchTool()` | Execute search with parsing, formatting, truncation, error handling |
| `formatErrorResponse()` | Format errors in German for user-facing output |

### Error Classes (client.ts)

- `RISAPIError` — Base error with statusCode
- `RISTimeoutError` — 30s timeout exceeded
- `RISParsingError` — JSON parsing failures, includes originalError

### Constants

- **Timeout**: 30,000ms (30 seconds)
- **Character limit**: 25,000 characters (formatting.ts `CHARACTER_LIMIT`)
- **Pagination**: 10/20/50/100 documents per page (mapped via `limitToDokumenteProSeite()` in types.ts)
- **Allowed document hosts**: `data.bka.gv.at`, `www.ris.bka.gv.at`, `ris.bka.gv.at` (SSRF protection in client.ts)

### Conventions

- **Language**: User-facing error messages are in **German**; tool descriptions and parameter `.describe()` text are in **English** (existing convention). Tool `title` (display name via `registerTool`) is German.
- **Imports**: Enforced order — builtin > external > internal > parent > sibling > index (alphabetized)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Honeyfield-Org/ris-mcp-ts](https://github.com/Honeyfield-Org/ris-mcp-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
