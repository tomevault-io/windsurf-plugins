---
trigger: always_on
description: TypeScript MCP server template with tools, resources, prompts, and session configuration.
---


# AGENTS.md

Welcome to the **Smithery TypeScript MCP Server Scaffold**!

This is the template project that gets cloned when you run `npx create-smithery`. It provides everything you need to build, test, and deploy a Model Context Protocol (MCP) server with Smithery.

## Table of Contents

- [Project Structure](#project-structure)
- [Quick Start Commands](#quick-start-commands)
- [smithery.yaml Configuration](#smitheryyaml-configuration)
- [Concepts](#concepts)
- [Development Workflow](#development-workflow)
- [Deployment & CI/CD](#deployment--cicd)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)
- [Community & Support](#community--support)

### Project Structure

```
your-server/
├── package.json           # Project dependencies and scripts
├── smithery.yaml          # Runtime specification (runtime: typescript)
├── src/
│   └── index.ts          # Main server implementation
└── README.md
```

## Quick Start Commands

```bash
# Run development server (streamable HTTP on port 8081)
# Opens interactive Smithery playground in your browser for testing
npm run dev          # or: bun run dev, pnpm run dev, yarn dev

# Run on a custom port
npm run dev -- --port 3000

# Kill existing process if port 8081 is in use
lsof -ti:8081 | xargs kill

# Build for production
npm run build        # or: bun run build, pnpm run build, yarn build
```

## smithery.yaml Configuration

The `smithery.yaml` file configures how your server runs. For this TypeScript setup, it only needs:

### Required Field

```yaml
runtime: typescript
```

This tells Smithery to use the TypeScript runtime for your server.

### Optional Fields

#### target (optional)

Specifies where your server runs and determines the transport protocol. Can be `local` or `remote`:

```yaml
runtime: typescript
target: remote    # Options: remote (default) or local
```

- `local`: Server runs on the user's machine using stdio transport. When published, bundled into `.mcpb` file for distribution
- `remote`: Server runs on Smithery's infrastructure using HTTP transport (default)

See [Transports](#transports) for more details on how this affects your server's communication protocol.

#### env (optional)

Environment variables to inject when running your server. Available for both runtime types:

```yaml
runtime: typescript
env:
  NODE_ENV: "production"
  DEBUG: "true"
  LOG_LEVEL: "info"
```

## Concepts

### Core Components: Tools, Resources, and Prompts

MCP servers expose three types of components that AI applications can interact with. Learn when to use each and how they work together to build powerful integrations.

#### Tools: Executable Functions

Tools are executable functions that AI applications can invoke to perform actions:

```typescript
// Add a tool
server.registerTool(
  "hello",
  {
    title: "Hello Tool",
    description: "Say hello to someone",
    inputSchema: { name: z.string().describe("Name to greet") },
  },
  async ({ name }) => {
    return {
      content: [{ type: "text", text: `Hello, ${name}!` }],
    }
  },
)
```

#### Resources: Read-Only Data Sources

Resources provide read-only data that gives AI applications context to work with. Unlike tools, resources do not perform actions—they simply return information:

```typescript
// Add a resource
server.registerResource(
  "hello-world-history",
  "history://hello-world",
  {
    title: "Hello World History",
    description: "The origin story of the famous 'Hello, World' program",
  },
  async uri => ({
    contents: [
      {
        uri: uri.href,
        text: '"Hello, World" first appeared in a 1972 Bell Labs memo by Brian Kernighan and later became the iconic first program for beginners in countless languages.',
        mimeType: "text/plain",
      },
    ],
  }),
)
```

#### Prompts: Reusable Message Templates

Prompts are predefined message templates that help structure conversations. Use them to guide AI applications toward consistent interaction patterns:

```typescript
// Add a prompt
server.registerPrompt(
  "greet",
  {
    title: "Hello Prompt",
    description: "Say hello to someone",
    argsSchema: {
      name: z.string().describe("Name of the person to greet"),
    },
  },
  async ({ name }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Say hello to ${name}`,
          },
        },
      ],
    }
  },
)
```

#### When to Use Each Component

- **Tools**: Perform actions (create/update/delete, API calls, computations, database operations)
- **Resources**: Provide read-only data (documentation, reference info, context without side effects)
- **Prompts**: Guide conversation patterns (reusable templates, multi-step workflows, consistent interactions)

### Session Configuration

Pass personalized settings to each connection—API keys, preferences, and user-specific configuration—without affecting other sessions.

**Why configuration matters:**
- **Multi-user support**: Different users have different API keys and settings
- **Security**: API keys stay session-scoped, not stored server-wide
- **Flexibility**: Users customize behavior at connection time without code changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aryankeluskar/polymarket-mcp](https://github.com/aryankeluskar/polymarket-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
