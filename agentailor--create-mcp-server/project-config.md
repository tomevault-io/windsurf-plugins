---
trigger: always_on
description: A CLI tool to scaffold new MCP (Model Context Protocol) server projects.
---

# @agentailor/create-mcp-server

A CLI tool to scaffold new MCP (Model Context Protocol) server projects.

## Project Structure

```
create-mcp-server/
├── src/
│   ├── index.ts                    # CLI entry point
│   ├── cli.ts                      # CLI argument parsing (Commander.js)
│   ├── cli.test.ts                 # Tests for CLI argument parsing
│   ├── interactive.ts              # Interactive prompt flow (prompts library)
│   ├── project-generator.ts        # Shared project generation logic
│   └── templates/
│       ├── common/                 # Shared template files
│       │   ├── package.json.ts     # package.json template (framework-aware)
│       │   ├── tsconfig.json.ts    # tsconfig.json template
│       │   ├── gitignore.ts        # .gitignore template
│       │   ├── env.example.ts      # .env.example template
│       │   └── templates.test.ts   # Tests for common templates
│       ├── deployment/             # Deployment configuration templates
│       │   ├── dockerfile.ts       # Dockerfile template
│       │   ├── dockerignore.ts     # .dockerignore template
│       │   ├── index.ts            # Barrel exports
│       │   └── templates.test.ts   # Tests for deployment templates
│       ├── sdk/                    # Official MCP SDK templates
│       │   ├── stateless/          # Stateless HTTP template
│       │   │   ├── server.ts       # MCP server definition template
│       │   │   ├── index.ts        # Barrel export + getIndexTemplate
│       │   │   ├── readme.ts       # README.md template
│       │   │   └── templates.test.ts
│       │   ├── stateful/           # Stateful HTTP template with OAuth option
│       │   │   ├── server.ts       # Re-exports from stateless
│       │   │   ├── index.ts        # Barrel export + getIndexTemplate
│       │   │   ├── readme.ts       # README.md template
│       │   │   ├── auth.ts         # OAuth authentication template
│       │   │   ├── auth.test.ts    # Tests for auth template
│       │   │   └── templates.test.ts
│       │   └── stdio/              # stdio transport template
│       │       ├── server.ts       # Re-exports from stateless
│       │       ├── index.ts        # Barrel export + getIndexTemplate (StdioServerTransport)
│       │       ├── readme.ts       # README.md template (for local clients)
│       │       └── templates.test.ts
│       └── fastmcp/                # FastMCP templates
│           ├── server.ts           # FastMCP server definition template
│           ├── index.ts            # Barrel export + getIndexTemplate
│           ├── readme.ts           # README.md template
│           └── templates.test.ts
├── dist/                           # Compiled output (generated)
├── docs/
│   └── oauth-setup.md              # OAuth setup guide for various providers
├── official-examples/              # Reference MCP server examples
├── package.json
├── tsconfig.json
└── README.md
```

## Development

```bash
# Install dependencies
npm install

# Build
npm run build

# Test locally (interactive mode)
node dist/index.js

# Test locally (CLI mode)
node dist/index.js --name=test-server
node dist/index.js --name=test-server --package-manager=pnpm --framework=fastmcp

# Run tests
npm test
npm run test:watch

# Lint
npm run lint
npm run lint:fix

# Format
npm run format
npm run format:check
```

## Publishing

```bash
npm publish --access public
```

## CLI Modes

The CLI supports two modes:

### Interactive Mode (default)
When run without arguments, prompts the user for all options:
```bash
npx @agentailor/create-mcp-server
```

### CLI Mode
When any `--arg` is provided, all options must be specified via arguments (or use defaults):
```bash
npx @agentailor/create-mcp-server --name=my-server [options]
```

**CLI Options:**
| Option | Short | Default | Values |
|--------|-------|---------|--------|
| `--name` | `-n` | (required) | alphanumeric, hyphens, underscores |
| `--package-manager` | `-p` | `npm` | npm, pnpm, yarn |
| `--framework` | `-f` | `sdk` | sdk, fastmcp |
| `--stdio` | — | `false` | flag; uses stdio transport instead of HTTP |
| `--template` | `-t` | `stateless` | stateless, stateful (HTTP only, ignored with --stdio) |
| `--oauth` | — | `false` | flag (sdk+stateful only, incompatible with --stdio) |
| `--no-git` | — | `false` | flag |

## Frameworks

### Official MCP SDK (default)

Uses the official `@modelcontextprotocol/sdk` package with Express.js for full control.

### FastMCP

Uses [FastMCP](https://github.com/punkpeye/fastmcp), a TypeScript framework built on top of the official SDK that provides a simpler, more intuitive API.

## Templates

### SDK Templates

#### sdk/stateless

A stateless streamable HTTP MCP server using the official SDK. Each request creates a new transport and server instance.

Features:
- Express.js with `StreamableHTTPServerTransport`
- No session management (new transport per request)
- Example prompt (`greeting-template`)
- Example tool (`start-notification-stream`)
- Example resource (`greeting-resource`)
- TypeScript configuration
- Environment variable support for PORT and ALLOWED_HOSTS

#### sdk/stateful

A stateful streamable HTTP MCP server with session management using the official SDK.

Features:
- Session tracking via `mcp-session-id` header

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentailor/create-mcp-server](https://github.com/agentailor/create-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
