---
trigger: always_on
description: Framelink MCP for Figma is a Model Context Protocol (MCP) server that gives AI coding tools (Cursor, etc.) access to Figma design data. It fetches Figma files/nodes via the Figma API, simplifies the response to include only relevant layout and styling information, and serves it to AI clients.
---

# Framelink MCP for Figma

Framelink MCP for Figma is a Model Context Protocol (MCP) server that gives AI coding tools (Cursor, etc.) access to Figma design data. It fetches Figma files/nodes via the Figma API, simplifies the response to include only relevant layout and styling information, and serves it to AI clients.

## Build & Development Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build with tsup (outputs to dist/)
pnpm dev              # Development mode with watch + auto-restart (HTTP)
pnpm dev:cli          # Development mode (stdio)
pnpm test             # Run Vitest tests
pnpm type-check       # TypeScript type checking only
pnpm lint             # ESLint
pnpm format           # Prettier formatting
pnpm inspect          # Run MCP inspector for debugging
```

### Running the Server

```bash
pnpm start            # HTTP mode (default port 3333)
pnpm start:cli        # stdio mode for MCP clients
```

### Running a Single Test

```bash
pnpm test -- path/to/test.ts
pnpm test -- --testNamePattern="pattern"
```

### Releasing

Releases are automated via [release-please](https://github.com/googleapis/release-please). On merge to `main`, release-please reads conventional commit prefixes (`fix:`, `feat:`, `feat!:`) and maintains a release PR. Merging the release PR publishes to npm via OIDC trusted publishing.

### PR Title Convention

PRs are squash-merged, so the PR title becomes the commit message that release-please parses. Always use [Conventional Commit](https://www.conventionalcommits.org/) prefixes in PR titles.

## Architecture

### Entry Points

- `src/bin.ts` — CLI entry point, calls `startServer()`
- `src/server.ts` — Server initialization, handles stdio vs HTTP mode selection
- `src/mcp-server.ts` — Library re-exports for external consumers (`createServer`, `startServer`, etc.)
- `src/index.ts` — Library exports (extractors, types)

### Transport Modes

The server supports two transports (configured in `src/server.ts`):

- **stdio** — For direct MCP client integration (activated with `--stdio` flag or `NODE_ENV=cli`)
- **StreamableHTTP** — Stateless HTTP transport at `/mcp` (also served at `/sse` for backward compatibility with existing client configs)

### Core Data Flow

1. **MCP Tools** (`src/mcp/tools/`) — Define tool schemas and handlers

   - `get_figma_data` — Fetches and simplifies Figma design data
   - `download_figma_images` — Downloads images from Figma

2. **Figma Service** (`src/services/figma.ts`) — API client for Figma REST API

   - Handles auth (Personal Access Token or OAuth)
   - Methods: `getRawFile()`, `getRawNode()`, `downloadImages()`

3. **Extractor System** (`src/extractors/`) — Transforms raw Figma API responses

   - `design-extractor.ts` — Entry point, parses API response and calls extractors
   - `node-walker.ts` — Recursive traversal applying extractors to each node
   - `built-in.ts` — Built-in extractors: `layoutExtractor`, `textExtractor`, `visualsExtractor`, `componentExtractor`
   - Extractors are composable; `allExtractors` combines all built-ins

4. **Transformers** (`src/transformers/`) — Convert specific Figma properties
   - `layout.ts` — Layout/positioning transforms
   - `style.ts` — Visual styling (fills, strokes)
   - `effects.ts` — Effects (shadows, blurs)
   - `text.ts` — Text content and styling
   - `component.ts` — Component metadata

### Configuration

`src/config.ts` handles CLI args and environment variables:

- `FIGMA_API_KEY` or `--figma-api-key` — Personal Access Token
- `FIGMA_OAUTH_TOKEN` or `--figma-oauth-token` — OAuth Bearer token
- `PORT` or `--port` — HTTP server port (default: 3333)
- `--json` — Output JSON instead of YAML
- `--skip-image-downloads` — Disable image download tool

### Path Alias

The codebase uses `~/` as an alias for `src/` (configured in tsconfig.json and vitest.config.ts).

## Philosophy

From CONTRIBUTING.md — important context for development:

1. **Unix Philosophy** — Tools should have one job and few arguments. Keep tools simple to avoid confusing LLMs.
2. **Focused Scope** — The server only handles "ingesting designs for AI consumption." Out of scope: image manipulation, CMS syncing, code generation, third-party integrations.
3. **Project-level Config** — Options unlikely to change between requests should be CLI arguments, not tool parameters.

## Quality

This codebase will outlive you. Every shortcut becomes someone else's burden. Every hack compounds into technical debt that slows the whole team down.

For each proposed change, examine the existing system and redesign it into the most elegant solution that would have emerged if the change had been a foundational assumption from the start.

You are not just writing code. You are shaping the future of this project. The patterns you establish will be copied. The corners you cut will be cut again.

Fight entropy. Leave the codebase better than you found it.

## Comment Policy

### Unacceptable Comments

- Comments that repeat what code does
- Commented-out code (delete it)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GLips/Figma-Context-MCP](https://github.com/GLips/Figma-Context-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
