---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
with code in this repository.

@doc/glossary.md
@doc/mcp-specification.md

## Architecture Overview

This is a Clojure implementation of the Model Context Protocol (MCP)
designed as a polylith-style modular architecture with component-based
organization:

- **Component Architecture**: Each major functionality is isolated in
  `components/` with its own `deps.edn`, src, and test directories
- **Projects Structure**: The `projects/server/` provides a deployable
  server artifact that composes the components
- **Namespace Convention**: All code uses the `mcp-clj` top-level
  namespace as specified in `design/namespaces.md`

### Core Components

- `json-rpc/` - JSON-RPC 2.0 server implementation with automatic
                EDN/JSON conversion
- `mcp-server/` - MCP protocol implementation with tools, prompts, and
                  resources support
- `mcp-client/` - MCP client implementation for connecting to MCP servers
- `http-server/` - HTTP adapter for serving JSON-RPC over HTTP
- `http/` - HTTP client utilities
- `tools/` - MCP tools registry and management
- `interop/` - Java SDK interoperability layer
- `client-transport/` - Client-side transport layer abstraction
- `server-transport/` - Server-side transport layer abstraction
- `in-memory-transport/` - In-memory transport for testing and local
                           communication
- `log/` - Logging infrastructure

### Base Implementations

- `sse-server/` - Server-Sent Events (SSE) transport server base
- `stdio-server/` - Standard I/O transport server base

Each operation for a server capability is implemented has a handler in
the mcp-server component in the `mcp-clj.mcp-server.core` namespace.
The implementation for implementing each capability is in the
`mcp-clj.mcp-server.<capability>` namespace.

Each operation for a server capability is exposed in the mcp-client
component in the `mcp-clj.mcp-client.core` namespace.  The
implementation for exposing each capability is in the
`mcp-clj.mcp-client.<capability>` namespace.

Concerns follow the same pattern as capabilities.

### Key Design Patterns

- **Handler Functions**: JSON-RPC handlers work with native EDN data
  structures; the server handles JSON conversion automatically
- **Registry Pattern**: Tools, prompts, and resources are managed
  through registry components that support dynamic updates with change
  notifications
- **Session Management**: MCP server maintains client sessions with
  initialization state and capabilities
- **Protocol Compliance**: Implements MCP specification version
  "2024-11-05" with proper version negotiation

## Common Development Commands

### Testing

This project uses a two-tier testing strategy to optimize development speed:

**Unit Tests (Default - Fast)**
- Run by default with `clj -M:kaocha:dev:test`
- Exclude integration tests (marked with `^:integ` metadata)
- Test pure functions and isolated components
- Typically complete in seconds

**Integration Tests (Slower)**
- Run with `clj -M:kaocha:dev:test --focus :integration`
- Include tests marked with `^:integ` metadata
- Start actual servers, external processes, or cross-process communication
- Examples: HTTP server tests, MCP client-server integration, Java SDK interop

```bash
# Run only fast unit tests (default)
clj -M:kaocha:dev:test --reporter kaocha.report/dots

# Run only integration tests (slower)
clj -M:kaocha:dev:test --focus :integration

# Run all tests (unit + integration)
clj -M:kaocha:dev:test --focus :unit :integration --reporter kaocha.report/dots

# Run specific namespace
clj -M:kaocha:dev:test --focus mcp-clj.mcp-client.tools-test

# Run tests with coverage (uncomment cloverage plugin in tests.edn)
clj -M:kaocha:dev:test --plugin kaocha.plugin/cloverage
```

**Test Classification Guidelines:**
- Mark tests with `^:integ` if they:
  - Start MCP servers or HTTP servers
  - Launch external processes or subprocesses
  - Use `with-test-server`, `with-http-test-env`, or similar macros
  - Test cross-process communication
- Keep unit tests fast and focused on isolated functionality

**MCP Client Test Patterns:**
- When testing with MCP clients, always wait for initialization to complete
  before sending requests:
  ```clojure
  (let [client (mcp-client/create-client {...})]
    (mcp-client/wait-for-ready client 5000)  ; Wait up to 5 seconds
    ;; Now safe to make requests
    @(mcp-client/call-tool client "tool-name" {...}))
  ```
- Do not directly deref `:initialization-future` - use `wait-for-ready`
  which provides proper error handling and timeout support

```clojure
;; Run tests from REPL after code changes
(require 'my.test.namespace :reload)
(clojure.test/run-tests 'my.test.namespace)

;; Run specific test
(require 'mcp-clj.mcp-server.version-test :reload)
(clojure.test/run-tests 'mcp-clj.mcp-server.version-test)
```

### Development REPL
```bash
# Start development REPL with all components loaded
clj -M:dev

# In REPL, after adding new dependencies to deps.edn:
(sync-deps)  ; Load new dependencies without restart
```

### Server Usage
```bash
# Start SSE server on port 3456
clj -M:sse-server

# Start STDIO server
clj -M:stdio-server
```

```clojure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hugoduncan/mcp-clj](https://github.com/hugoduncan/mcp-clj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
