---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

### Building
```bash
npm run build                  # Production build (minified, bundled)
npm run build:dev             # Development build (external packages, debug logging)
npm run build:desktop         # Desktop variant build
npm run build:combined        # Combined variant build
npm run build:docker          # Build Docker image
```

### Running the Server
```bash
npm run start:http            # Start HTTP server (default port 3927)
npm run start:http:apm        # Start with APM telemetry tracing enabled
npm run start:http:docker     # Run via Docker container
```

### Testing
```bash
npm test                      # Unit tests (src/**/*.test.ts)
npm run test:e2e              # End-to-end tests
npm run test:eval             # Evaluation tests
npm run test:oauth:embedded   # OAuth embedded tests
npm run test:oauth:tableau    # OAuth Tableau tests (Playwright)
npm run coverage              # Unit test coverage report
```

### Linting & Formatting
```bash
npm run lint                  # Run ESLint
npm run lint:fix              # Auto-fix ESLint issues
```

### MCP Inspector
```bash
npm run inspect               # Build + inspect with MCP Inspector (stdio)
npm run inspect:http          # Run HTTP server + inspector
npm run inspect:docker        # Build Docker + inspect
```

### Running Individual Tests
```bash
npx vitest run src/path/to/file.test.ts              # Run specific unit test
npx vitest run src/path/to/file.test.ts -t "pattern"  # Run tests matching pattern
npx playwright test tests/oauth/tableau-authz         # Run specific Playwright test
```

## Architecture

### Build Variants
The project supports multiple build variants via `src/scripts/build.ts`:
- **default**: Standard HTTP/stdio server
- **desktop**: Desktop-specific variant with ext-apps integration
- **combined**: Both web and desktop tools

Build variants use esbuild with conditional code via `globalIdentifiers` to customize the final bundle.

### Transport Modes
Two MCP transport modes defined in `src/transports.ts`:
- **stdio**: Standard input/output (for Claude Desktop, npx usage)
- **http**: HTTP server with Express (for multi-user deployments)

### Authentication Types
Configured via `AUTH` env var (see `src/config.ts`):
- **pat**: Personal Access Token (PAT_NAME + PAT_VALUE)
- **uat**: User Access Token with JWT
- **direct-trust**: Connected App direct trust (CONNECTED_APP_* vars)
- **oauth**: OAuth 2.0 flow (embedded or Tableau authz server)

### Core Server Architecture

#### Entry Point (`src/index.ts`)
- Loads dotenv configuration
- Initializes Tableau server info (via `getTableauServerInfo`)
- Creates either stdio or HTTP transport based on `TRANSPORT` env var
- Sets up notification level and file logging if configured

#### Web Server (`src/server.web.ts` + `src/server/express.ts`)
- `WebMcpServer` extends base `Server` class
- Registers web tools from `src/tools/web/tools.ts`
- Express server handles:
  - OAuth middleware (embedded or Tableau provider)
  - Passthrough auth middleware (`X-Tableau-Auth` header)
  - Session management (unless `DISABLE_SESSION_MANAGEMENT=true`)
  - CORS configuration
  - Latency metrics
  - MCP StreamableHTTPServerTransport

#### Desktop Server (`src/server.desktop.ts`)
- Desktop variant uses `@modelcontextprotocol/ext-apps` for discovery
- Integrates with `DesktopDiscoverer` to find local Tableau Desktop instances
- Tools are proxied to desktop via `DesktopToolExecutor`

### Tool Architecture

#### Web Tools (`src/tools/web/`)
Each tool is a subdirectory containing:
- `tool.ts`: Tool definition extending `WebTool` class
- `*.test.ts`: Unit tests
- Tool implements `paramsSchema` (Zod) and `callback` function

Key tools include:
- `queryDatasource`: Query Tableau datasources
- `contentExploration`: Search workbooks, views, datasources
- `getDatasourceMetadata`: Retrieve datasource schema/fields
- `pulse`: Pulse-related operations
- `projects`: Project management
- `adminInsights`: Admin insights queries

#### Desktop Tools (`src/tools/desktop/`)
Desktop-specific tools that interact with local Tableau Desktop instances.

#### Tool Context (`src/tools/web/toolContext.ts`)
`TableauWebRequestHandlerExtra` provides:
- Config with request overrides
- Tableau auth info (userId, siteId, scopes)
- Server reference
- User/site LUID accessors

### SDK Layer (`src/sdks/tableau/`)
Wraps Tableau REST API:
- `restApi.ts`: Main API client using Zodios
- `apis/`: REST API endpoint definitions
- `methods/`: Higher-level method wrappers
- `types/`: TypeScript types for API responses

### Configuration (`src/config.ts`)
`Config` class extends `BaseConfig` with environment variables:
- Server connection (SERVER, SITE_NAME)
- Auth credentials (PAT_*, UAT_*, CONNECTED_APP_*)
- OAuth settings (OAUTH_*)
- Feature flags (ENABLE_MCP_SITE_SETTINGS, ENABLE_PASSTHROUGH_AUTH)
- Telemetry (TELEMETRY_PROVIDER, PRODUCT_TELEMETRY_ENDPOINT)

### Feature Gates (`src/features/`)
Provider-based feature flag system (mirrors telemetry provider pattern):
- **Server provider** (default): Loads `features.json` from filesystem (on-prem)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tableau/tableau-mcp](https://github.com/tableau/tableau-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
