---
trigger: always_on
description: This file provides guidance to AI coding assistants (Claude Code, Cursor, GitHub Copilot, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants (Claude Code, Cursor, GitHub Copilot, etc.) when working with code in this repository.

## Project Overview

Moclojer is an HTTP mock server written in Clojure that generates mock APIs from YAML, EDN, OpenAPI, or Postman Collection specifications. It supports both HTTP endpoints and WebSocket connections.

**⚠️ IMPORTANT: Code changes REQUIRE documentation updates!** See [Documentation Requirements](#documentation-requirements) section.

## Development Commands

```bash
# Run the server
clj -M:run

# Run all tests
clj -M:test

# Run a specific test
clj -M:test -n com.moclojer.external-body.excel-test

# Lint the source code
clj -M:lint

# Start nREPL for interactive development
clj -M:nrepl

# Build JAR file
clj -A:dev -M --report stderr -m com.moclojer.build

# Deploy to Clojars (requires CLOJARS_USERNAME and CLOJARS_PASSWORD env vars)
clj -X:deploy-clojars
```

## Architecture

### Core Flow

1. **Entry Point** (`com.moclojer.core`): CLI parsing via babashka.cli, loads config and starts server
2. **Adapters** (`com.moclojer.adapters`): Transforms CLI args/env into config, generates routes from config
3. **Router** (`com.moclojer.router`): Smart router that detects spec type (moclojer vs OpenAPI vs Postman) and delegates
4. **Server** (`com.moclojer.server`): http-kit server with reitit router, includes file watcher for hot reload

### Spec Processing

- **`com.moclojer.specs.moclojer`**: Converts moclojer YAML/EDN spec to reitit routes. Handles HTTP endpoints, WebSocket endpoints, template rendering (Selmer), and webhooks
- **`com.moclojer.specs.openapi`**: Converts OpenAPI v3 spec to moclojer format (path conversion: `{id}` -> `:id`)
- **`com.moclojer.specs.postman`**: Converts Postman Collection v2.1 to moclojer format. Processes nested folders, extracts response examples, converts path variables

### Key Components

- **Template Engine**: Uses Selmer for dynamic responses. Variables: `{{path-params.name}}`, `{{query-params.name}}`, `{{json-params.field}}`
- **External Body**: Supports JSON and XLSX files as response sources (`com.moclojer.external-body.*`). Global folder configuration available via `- external-body: { folder: path }` in spec.
- **Middleware**: Rate limiting (`middleware/rate_limit.clj`) and latency simulation (`middleware/latency.clj`)
- **Webhooks**: Async webhook calls with configurable delay and conditions (`com.moclojer.webhook`)
- **File Watcher**: Hot reload on config changes (disabled in GraalVM native image)

### WebSocket Support

WebSocket endpoints use http-kit's `as-channel` with pattern matching for messages. Config structure:
```yaml
- websocket:
    path: /ws/echo
    on-connect:
      response: "welcome"
    on-message:
      - pattern: "ping"
        response: "pong"
```

## Configuration

- Default config location: `~/.config/moclojer.yml` (XDG_CONFIG_HOME)
- Environment variables: `CONFIG`, `MOCKS`, `PORT`, `HOST`, `SENTRY_DSN`
- Supports multi-domain routing via `host` field in endpoint config

## Testing

Tests are in `test/com/moclojer/`. Resource files for tests are in `test/com/moclojer/resources/`.

## Code Style

- Namespaces follow `com.moclojer.*` convention
- Private functions use `^:private` metadata or `-` prefix (e.g., `-main`)
- Use threading macros (`->`, `->>`) for transformation pipelines
- Docstrings required for all public functions
- Logging via `com.moclojer.log/log`, never `println`
- kebab-case for functions and variables, PascalCase avoided

## Patterns

- **Atom for state**: Router uses atom for hot-reload (`*router`)
- **Specs as data**: YAML/EDN config → Clojure map → reitit routes
- **Middleware chain**: Rate limit → Latency → Parameters → Handler
- **Template rendering**: Selmer interpolates variables in responses
- **Async webhooks**: `core.async/go` blocks for delayed HTTP calls

## Architecture Decisions

- **http-kit** over ring-jetty: native WebSocket and async support
- **reitit** over compojure: data-driven routing, better for dynamic generation
- **Selmer** for templates: familiar syntax (Jinja2-like), safe by default
- **No database**: mock server is stateless by design
- **XDG compliance**: respects `XDG_CONFIG_HOME` for config location

## Extension Points

- **New spec format**: add parser in `com.moclojer.specs/`, update `router/smart-router`
- **New external-body provider**: add in `com.moclojer.external-body/`, update `type-identification`
- **New middleware**: add in `com.moclojer.middleware/`, register in `server.clj` middleware chain

## Documentation Requirements

**CRITICAL: Always update documentation when making changes!**

### When to Update Documentation

Update documentation in `docs/` for ANY of these changes:

1. **New Features**
   - Add tutorial/guide in `docs/getting-started/` or `docs/advanced/`
   - Update `docs/SUMMARY.md` to include new page
   - Update `docs/README.md` if it affects learning path
   - Add examples in `examples/` directory

2. **New Configuration Options**
   - Update `docs/reference/configuration-spec.md`
   - Add examples in relevant tutorial pages
   - Update CLI reference if applicable

3. **API Changes**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moclojer/moclojer](https://github.com/moclojer/moclojer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
