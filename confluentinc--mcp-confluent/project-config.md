---
trigger: always_on
description: MCP (Model Context Protocol) server that exposes Kafka-ecosystem and Confluent tooling to AI
---

# mcp-confluent

## Project Overview

MCP (Model Context Protocol) server that exposes Kafka-ecosystem and Confluent tooling to AI
assistants. The tool surface splits into two groups:

- **Kafka-protocol tools** that work against any Apache Kafka®-compatible cluster or Schema
  Registry (e.g., topic CRUD, producing and consuming messages, schema management).
- **Confluent Cloud-specific tools** that wrap CCloud REST APIs (e.g., Flink, Tableflow,
  billing), enabled only when the relevant service block (`confluent_cloud:`, `flink:`,
  `tableflow:`) is present in a connection's resolved config (from YAML or, during the
  migration, legacy env vars + CLI args).

Built with TypeScript, Node.js ≥22, and the `@modelcontextprotocol/sdk`. Ships as an npm package
and a Docker image; supports stdio, Streamable HTTP, and (for backwards compatibility with older
MCP clients) the legacy HTTP+SSE transport.

These instructions exist to keep GitHub Copilot PR reviews focused on the invariants that matter in
this codebase. Author-facing guidance (how to scaffold a tool, run the inspector, etc.) lives in
`CONTRIBUTING.md` and `CLAUDE.md` — refer to those rather than repeating them here.

## Architecture (what reviewers need to know)

### Entry flow

At startup: `parseCliArgs()` → `initEnv()` → branch on whether `-c <path>` was supplied:
`loadConfigFromYaml(path, process.env)` for the YAML path, or `buildConfigFromEnvAndCli(env, ...)`
for the legacy env+CLI path. Both return a single `MCPServerConfiguration`. Then
`ServerRuntime.fromConfig()` constructs one `DirectClientManager` per connection → iterates
`ToolName` enum to build the enabled tool set → registers tools on `McpServer` → starts transports.

YAML (`-c <path>`) is the preferred path. The env-var-only path is legacy: parity remains for a
single connection in this release, but it's slated for a startup warning in a near-future release
and removal a release or two later. Don't approve new code that reaches back into the legacy
synthesizer or assumes env-only configuration as the canonical model. User-facing version of this
story: `CONFIGURATION.md` at the repo root.

**Tools are auto-enabled/disabled** at startup based on which service blocks are present in each
connection's resolved config (which can come from a YAML file or, for backwards compatibility
during the migration, from env vars + CLI args). Each handler declares its requirement via a
`predicate` property referencing a named export from
`src/confluent/tools/connection-predicates.ts`; `BaseToolHandler` derives
`enabledConnectionIds(runtime)` and `connectionVerdicts(runtime)` from it. An empty result
disables the tool. Both methods are `@final` — handlers must not override them.

### Key layers

- **`src/config/`** — Configuration core. `models.ts` defines `MCPServerConfiguration` (a Zod
  schema with per-service connection blocks: `kafka`, `flink`, `schema_registry`,
  `confluent_cloud`, `tableflow`, `telemetry`). `index.ts` exposes `loadConfigFromYaml()` for
  the `-c <path>` branch (parsing, `${VAR}` interpolation via `interpolation.ts`, and Zod
  validation). `env-config.ts` exports `buildConfigFromEnvAndCli()` for the legacy env-var + CLI
  path. Both return an `MCPServerConfiguration`.

- **`src/confluent/tools/`** — Tool system core:
  - `tool-name.ts` — `ToolName` enum; every tool has an entry here.
  - `base-tools.ts` — `BaseToolHandler` abstract class all handlers extend (directly or via a
    domain subclass like `FlinkToolHandler` or `TableflowToolHandler`).
  - `connection-predicates.ts` — predicate vocabulary handlers use to express enablement:
    `hasKafka`, `hasFlink`, `hasSchemaRegistry`, `hasConfluentCloud`, `hasTableflow`,
    `hasTelemetry`, `hasKafkaRestWithAuth`, and conjunctions like `hasCCloudCatalogSupport`.
    `connectionIdsWhere(connections, predicate)` is the canonical way to call them.
  - `tool-registry.ts` — `ToolHandlerRegistry.handlers` maps `ToolName` → handler instance.
    Wiring must be complete here for a tool to exist.
  - `handlers/<domain>/` — organized by service (e.g., `kafka/`, `schema/`, `flink/`); new
    handlers go under the matching domain or a new one if no fit exists. Some domains expose an
    intermediate base class (e.g., `flink-tool-handler.ts`, `tableflow-tool-handler.ts`) that
    implements `enabledConnectionIds()` once for the whole domain.

- **`src/confluent/client-manager.ts`** — public client-manager contracts (`ClientManager` and
  its constituent interfaces).
- **`src/confluent/base-client-manager.ts`** — abstract `BaseClientManager` owning every typed
  `openapi-fetch` REST client and the Schema Registry SDK client. No native Kafka broker.
- **`src/confluent/direct-client-manager.ts`** — concrete `DirectClientManager` adding api-key
  Kafka admin/producer/consumer via `@confluentinc/kafka-javascript`.

- **`src/confluent/openapi-schema.d.ts`** — generated from `openapi.json` via
  `npm run generate:openapi-types` (openapi-typescript). Never hand-edited.

- **`src/confluent/node-deps.ts`** — namespace-object wrapper around Node builtins, third-party
  constructors, and env access. ESM named imports are read-only from outside the defining

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [confluentinc/mcp-confluent](https://github.com/confluentinc/mcp-confluent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
