---
trigger: always_on
description: <--help Workspace-specific instructions for GitHub Copilot -->
---

<--help Workspace-specific instructions for GitHub Copilot -->

# Komodo MCP Server

An MCP (Model Context Protocol) server that enables AI assistants to interact with [Komodo Container Manager](https://komo.do) for Docker container management, server orchestration, stack lifecycle, and deployment operations.

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| TypeScript | ^5.0.0 | Strict Mode, ES2022 target |
| Node.js | ≥20.0.0 | Runtime |
| `mcp-server-framework` | ^1.1.0 | MCP protocol layer, transports, logging, errors |
| `komodo_client` | — | Komodo API bindings (read/execute operations) |
| Zod | via framework | Runtime schema validation |
| Docker | node:22-alpine | Multi-stage production builds |

## Project Structure

```
src/
├── index.ts              # Entry point — createServer() + lifecycle hooks
├── client.ts             # KomodoClient wrapper + connection monitoring
├── config/
│   ├── index.ts          # Barrel export
│   ├── env.ts            # Zod schema for env vars + config file section
│   ├── descriptions.ts   # PARAM_DESCRIPTIONS, RESPONSE_ICONS, constants
│   ├── version.ts        # SERVER_NAME, SERVER_VERSION
│   └── tools.config.ts   # Tool-specific defaults (log tails, timeouts)
├── errors/
│   ├── index.ts          # Barrel export
│   ├── classes.ts        # ApiError, ConnectionError, AuthenticationError
│   ├── factory.ts        # AppErrorFactory (extends FrameworkErrorFactory)
│   ├── messages.ts       # AppMessages registry + getAppMessage()
│   └── extraction.ts     # Error parsing from komodo_client responses
├── tools/
│   ├── index.ts          # Side-effect imports (auto-registration)
│   ├── config.ts         # komodo_configure, komodo_health_check
│   ├── container.ts      # Container operations (10 tools)
│   ├── server.ts         # Server operations (6 tools)
│   ├── stack.ts          # Stack lifecycle (13 tools)
│   ├── deployment.ts     # Deployment operations (13 tools)
│   ├── terminal.ts       # Terminal exec (4 tools)
│   ├── user.ts           # User metadata (3 tools)
│   └── schemas/          # Shared Zod schemas per domain
└── utils/
    ├── index.ts           # Barrel export
    ├── api-helpers.ts     # requireClient(), wrapApiCall(), checkCancelled()
    ├── response-formatter.ts  # formatActionResponse(), formatListHeader()
    ├── resource-link.ts   # tryRegisterResource() — registers payload in DynamicResourceRegistry, returns ResourceLinkSpec | null
    ├── markdown.ts        # Domain-specific Markdown renderers (containers, servers, stacks, deployments)
    ├── polling.ts         # wrapExecuteAndPoll() for long-running operations
    └── polyfills.ts       # Node.js polyfills for komodo_client
```

## Key Patterns

### Framework Dependency

All MCP infrastructure comes from `mcp-server-framework`:

```typescript
import { createServer, defineTool, text, z, logger } from "mcp-server-framework";
```

This project provides only Komodo-specific logic: API client, tool definitions, error classes, and configuration.

### Tool Definition

Tools use `defineTool()` with auto-registration via side-effect imports:

```typescript
import { defineTool, text, z } from "mcp-server-framework";
import { requireClient, wrapApiCall } from "../utils/index.js";

defineTool({
  name: "komodo_container_start",
  description: "Start a stopped container",
  input: z.object({ server: z.string(), container: z.string() }),
  annotations: { readOnlyHint: false, destructiveHint: false },
  handler: async (args, { abortSignal }) => {
    const komodo = requireClient();
    const result = await wrapApiCall(
      "startContainer",
      () => komodo.client.execute("StartContainer", { server: args.server, container: args.container }),
      abortSignal,
    );
    return text(formatActionResponse({ ... }));
  },
});
```

### Structured + Resource Link Response

Inspect/info/logs tools return `structured()` with an optional dynamic resource link for out-of-band retrieval:

```typescript
import { defineTool, structured, z } from "mcp-server-framework";
import { tryRegisterResource } from "../utils/index.js";

defineTool({
  name: "komodo_container_inspect",
  handler: async (args, { abortSignal, sessionId }) => {
    const komodo = requireClient();
    const data = await wrapApiCall("inspect", () => komodo.client.read(...), abortSignal);
    const link = tryRegisterResource({
      sessionId,
      category: "container-inspect",
      content: JSON.stringify(data, null, 2),
      mimeType: "application/json",
      name: `Inspect: ${args.container}`,
    });
    return structured(data, {
      text: renderContainerInspect(data),
      ...(link && { links: [link] }),
    });
  },
});
```
```

### Error Handling

Centralized error creation via `AppErrorFactory`:

```typescript
import { AppErrorFactory } from "../errors/index.js";

throw AppErrorFactory.notFound.server("my-server");
throw AppErrorFactory.api.requestFailed("timeout exceeded");
throw AppErrorFactory.auth.invalidCredentials();
```

Messages via `getAppMessage()` with interpolation:

```typescript
import { getAppMessage } from "../errors/index.js";
getAppMessage("API_REQUEST_FAILED_REASON", { reason: "timeout" });
```

### API Call Wrapping


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MP-Tool/komodo-mcp-server](https://github.com/MP-Tool/komodo-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
