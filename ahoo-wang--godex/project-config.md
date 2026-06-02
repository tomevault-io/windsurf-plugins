---
trigger: always_on
description: Coding-agent instructions for the GodeX repository.
---

# AGENTS.md - GodeX

Coding-agent instructions for the GodeX repository.

## Build And Run

```bash
bun install                  # Install dependencies
bun run dev                  # Dev server with hot reload on port 13145
bun run start                # Start server from source
bun run build                # Build a binary for the current platform
bun run compile:all          # Cross-compile all supported platform packages
godex init                   # Create a godex.yaml interactively
godex serve --config ./godex.yaml
godex config check --config ./godex.yaml
godex config print --config ./godex.yaml
```

The runtime config default port is `5678`; `bun run dev` explicitly uses port `13145`.

## Testing

```bash
bun run typecheck            # tsc --noEmit
bun run lint                 # biome check src
bun run lint:fix             # biome check --write src
bun run format               # biome format --write src
bun run test                 # Unit + integration tests; excludes src/e2e
bun test src/bridge/tools/tool-plan.test.ts
bun run test:e2e             # Mocked upstream E2E tests
bun run test:zhipu           # Live Zhipu tests; needs ZHIPU_API_KEY
bun run test:coverage        # Coverage for non-e2e tests
bun run check                # typecheck + lint + test
bun run ci                   # typecheck + biome ci + test + e2e
```

Run `bun run check` before committing code. Run `bun run test:e2e` when request routing, providers, sessions, traces, or stream behavior changes.

## Project Structure

```text
src/
  cli/          Commander CLI, commands, init wizard, runtime config
  config/       godex.yaml parsing, defaults, validation, env interpolation
  context/      ApplicationContext and request-scoped ResponsesContext
  bridge/       Provider-agnostic Responses-to-Chat kernel
  providers/    Provider registry, specs, clients, hooks, protocol DTOs
  responses/    Sync and streaming orchestration pipelines
  server/       Bun routes: /health, /v1/models, /v1/responses
  resolver/     Model selector and alias resolution
  session/      Memory and SQLite previous_response_id stores
  trace/        SQLite trace records for requests, usage, events, errors
  logger/       LogTape-based structured logging
  error/        GodeXError hierarchy and domain codes
  protocol/     OpenAI protocol type definitions
  tools/        Codex built-in tool definitions
  testing/      Shared test provider utilities
```

Generated or external directories such as `node_modules/`, `dist/`, and platform build output should not be edited by hand.

## Runtime Flow

```text
CLI
  -> ApplicationContext
  -> Bun server
  -> POST /v1/responses
  -> parse request
  -> create ResponsesContext
  -> ModelResolver
  -> ResponseSessionStore chain lookup
  -> Registrar resolves ProviderEdge
  -> ResponsesBridgeRuntime
  -> ProviderExchange builds provider request
  -> ProviderEdge calls upstream Chat Completions API
  -> bridge/response or bridge/stream reconstructs Responses output
  -> trace, logging, session persistence
```

`/v1/responses` is the main compatibility endpoint. `/v1/models` exposes configured model aliases. `/health` reports registered and unsupported providers.

## Bridge Kernel

`src/bridge/` owns shared Responses-to-Chat behavior. Keep provider-agnostic policy here.

- `compatibility/` plans supported, degraded, ignored, and rejected request features.
- `request/` normalizes Responses input and session history into Chat Completions messages.
- `tools/` plans tool declarations, `tool_choice`, degradation, identity mapping, and call restoration.
- `output/` plans structured-output contracts and validates strict downgraded JSON output.
- `response/` reconstructs sync `ResponseObject` results from provider responses.
- `stream/` maps provider deltas into Responses SSE events through a state machine.
- `provider-spec/` defines `ProviderSpec`, `ProviderEdge`, provider constants, and package shape checks.
- `finish-reason/` maps provider finish reasons to Responses terminal states.

Do not duplicate compatibility decisions in provider hooks. Provider hooks should expose protocol differences; the bridge decides support, downgrade, rejection, and diagnostics.

## Responses Pipelines

`src/responses/` owns orchestration around the bridge kernel.

- `ProviderExchange` builds provider requests, records trace request/event rows, and calls `ProviderEdge`.
- `SyncRequestPipeline` reconstructs the final `ResponseObject`, validates output contracts, records usage, logs diagnostics, and persists sessions.
- `StreamPipeline` translates provider SSE chunks to Responses SSE events, validates terminal output, logs usage, persists sessions, records trace events, and emits compatibility diagnostics.
- `stream-transforms/` contains composable `TransformStream` stages.

The stream pipeline order matters: provider events are bridged first, output contracts are validated before logging and persistence, then SSE encoding happens in the server route.

## Provider Pattern

Each built-in provider uses a compact ProviderSpec package:

```text
src/providers/<name>/
  spec.ts       Capabilities, endpoint, auth, tool codec, accessors, hooks
  client.ts     create<Name>ProviderEdge(config)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ahoo-Wang/GodeX](https://github.com/Ahoo-Wang/GodeX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
