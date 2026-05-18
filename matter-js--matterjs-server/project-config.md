---
trigger: always_on
description: A Matter.js-based controller with a Python Matter Server-compatible WebSocket interface, designed for Home Assistant integration. The server listens on `localhost:5580/ws` and speaks the same protocol as the Python Matter Server, making it a drop-in replacement for Home Assistant's Matter integration.
---

# GitHub Copilot Instructions

## Project Overview

A Matter.js-based controller with a Python Matter Server-compatible WebSocket interface, designed for Home Assistant integration. The server listens on `localhost:5580/ws` and speaks the same protocol as the Python Matter Server, making it a drop-in replacement for Home Assistant's Matter integration.

## Build, Test & Lint Commands

```bash
# Install and full build
npm i

# Incremental build
npm run build

# Clean rebuild
npm run build-clean

# Run all tests
npm test

# Run a single test file
npx mocha --require tsx path/to/SomeTest.ts

# Lint
npm run lint
npm run lint-fix

# Format (rewrites files in-place — run BEFORE lint/build)
npm run format
npm run format-verify
```

> **MANDATORY — no exceptions:** After every code change, always run these four commands **in this exact order** before considering the work done:
> ```bash
> npm run format   # rewrites files in-place — MUST be first
> npm run lint
> npm run build
> npm test
> ```
> Skipping `format` is the most common mistake — oxfmt rewrites files in-place and the build/lint validate the formatted output. If you skip it, CI will fail.

**CI enforces formatting and linting.** The `check-and-lint` job in `build-test.js.yml` runs `npm run format-verify` and `npm run lint` and all other CI jobs (`test`, `build-non-linux`) are gated on it. PRs will fail if either check does not pass.

### Running the server

```bash
# Start the server (requires a prior build)
npm run server

# With options
npm run server -- --storage-path data --primary-interface en0 --bluetooth-adapter 0
```

### Debugging

```bash
# Send a WebSocket command to a running server
npm run send-command -- ws://localhost:5580/ws server_info
npm run send-command -- ws://localhost:5580/ws get_node '{"node_id": 1}'
npm run send-command -- ws://localhost:5580/ws device_command '{"node_id":1,"endpoint_id":1,"cluster_id":6,"command_name":"toggle","payload":{}}'
```

### Python client

```bash
# Install
npm run python:install

# Unit tests (fast, no server needed)
npm run python:test

# Integration tests (requires a running server on localhost:5580)
npm run python:test-integration
```

## Monorepo Structure

npm workspaces with six packages:

| Package | Name | Role |
|---------|------|------|
| `packages/tools` | `@matter/tools` | Private build infrastructure (esbuild + TSC). Provides `matter-build`, `matter-run`, `matter-version` binaries. **Never published.** |
| `packages/custom-clusters` | `@matter-server/custom-clusters` | Vendor-specific Matter cluster definitions (Eve, Inovelli, Heiman, etc.) registered into the Matter model at startup |
| `packages/ws-controller` | `@matter-server/ws-controller` | Core library: wraps `@project-chip/matter.js`, exports `MatterController`, `ControllerCommandHandler`, `WebSocketControllerHandler`, `ConfigStorage` |
| `packages/ws-client` | `@matter-server/ws-client` | WebSocket client library (used by the dashboard and for external consumers) |
| `packages/dashboard` | `@matter-server/dashboard` | Web UI (Lit + Material Web + Rollup) |
| `packages/matter-server` | `matter-server` | Main entry point: Express HTTP server, wires everything together |

## Architecture

### Server startup flow
`MatterServer.ts` → CLI options → Logger setup → `ConfigStorage` + `MatterController` → `WebServer` with:
- `WebSocketControllerHandler` — Python Matter Server-compatible WS API on `/ws`
- `StaticFileHandler` — serves dashboard assets
- `HealthHandler` — `/health` endpoint

Custom clusters are registered at import time via `import "@matter-server/custom-clusters"` in `MatterServer.ts`. The registration in `custom-clusters/src/register.ts` pushes each cluster definition into `Matter` and `ClusterRegistry`.

### WebSocket Protocol (Schema version 11)
Messages follow the Python Matter Server protocol. Key commands: `start_listening`, `commission_with_code`, `commission_on_network`, `device_command`, `read_attribute`, `write_attribute`, `subscribe_attribute`, `get_nodes`, `get_node`, `server_info`, `diagnostics`.

Events emitted to all clients: `node_added`, `node_updated`, `node_removed`, `attribute_updated`.

`start_listening` response is not logged (see `skipMessageContentInLogFor` in `WebSocketControllerHandler.ts`).

### Attribute path format
All `MatterNode.attributes` keys and attribute event paths use `"endpointId/clusterId/attributeId"` as decimal strings (e.g., `"0/40/4"` = endpoint 0, BasicInformation cluster, SoftwareVersion). Wildcards (`*`) are supported in read/subscribe commands.

### BigInt handling
The WS protocol uses a custom JSON serialiser (`toBigIntAwareJson`) and parser (`parseBigIntAwareJson`). Large numbers (≥15 digits, exceeding `MAX_SAFE_INTEGER`) are transmitted as raw decimal numbers (not quoted). Use these helpers — not `JSON.stringify/parse` — whenever serialising WS messages.

### Event broadcasting rules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matter-js/matterjs-server](https://github.com/matter-js/matterjs-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
