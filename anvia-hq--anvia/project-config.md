---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## Project Shape

Anvia is a TypeScript pnpm workspace for provider-agnostic AI runtime primitives,
provider adapters, vector stores, observability integrations, Studio, a cookbook,
and internal verification workspaces.

Workspace packages are declared in `pnpm-workspace.yaml`:

- `packages/*`
- `cookbook`
- `tests/*/*`

Use `pnpm` from the repository root. The repo declares `pnpm@11.0.4`.

## Before Editing

- Check `git status --short --branch` before making changes.
- Read the package, cookbook, or test workspace files you are about to change; follow local patterns.
- Do not edit `dist/`, coverage output, `node_modules/`, or other build artifacts by hand.
- Keep changes scoped. Avoid mixing dependency updates, generated artifact churn,
  formatting-only changes, and feature work unless they are required for the same
  change.
- Never commit secrets, API keys, private prompts, customer data, credentials,
  or trace payloads.

## Common Commands

Run from the repository root:

```sh
pnpm install
pnpm check
pnpm check:fix
pnpm format
pnpm typecheck
pnpm test
pnpm build
```

Prefer package-scoped commands while iterating:

```sh
pnpm --filter @anvia/core typecheck
pnpm --filter @anvia/core test
pnpm --filter @anvia/core build

pnpm --filter @anvia/openai typecheck
pnpm --filter @anvia/openai test

pnpm --filter @anvia/studio typecheck
pnpm --filter @anvia/studio test
pnpm --filter @anvia/studio build
```

CI builds `@anvia/core` first, then the remaining packages:

```sh
pnpm --filter @anvia/core build
pnpm --filter './packages/**' --filter '!@anvia/core' build
pnpm --filter './packages/**' typecheck
pnpm --filter './packages/**' test
```

## Formatting And Style

- TypeScript is strict, ESM, target `ES2022`, with `moduleResolution: "Bundler"`.
- Oxlint owns linting and Oxfmt owns formatting. Use repo scripts instead of invoking them ad hoc.
- Oxfmt uses 2-space indentation, double quotes, semicolons, trailing commas for
  JavaScript/TypeScript, and 100-column line width.
- Match each package's import specifier style. Some packages use extensionless
  local imports; many adapter packages use `.js` in source imports.
- Keep public APIs explicit and typed. Prefer runtime validation at tool,
  extraction, provider response, and external input boundaries.

## Package Boundaries

Package source lives in `src/`, tests usually live in `test/`, and build output
goes to `dist/`.

Most publishable packages expose `./dist/index.js` and `./dist/index.d.ts`.
`@anvia/core` has many subpath exports; when adding or moving a public entrypoint:

1. Add or update the source file.
2. Update the package `build` command if a new tsup entry is needed.
3. Update `exports` in the package `package.json`.
4. Update public re-exports from the relevant `src/index.ts` or subpath index.
5. Add or update tests.
6. Update package documentation and note any external documentation impact when public symbols
   change.

Keep provider-specific SDKs out of `@anvia/core`. Provider behavior belongs in
`packages/provider-*`; vector-store behavior belongs in `packages/vector-*`;
embedding adapter behavior belongs in `packages/embedding-*`; Studio runtime/UI
behavior belongs in `packages/tool-studio`.

## Package Map

- `packages/core`: core runtime for agents, completion, tools, hooks, request
  runtime, streaming, UI messages, extractors, pipelines, evals, embeddings,
  loaders, MCP, memory, model listing, observability, redaction, skills, transcription,
  audio/image generation, and vector-store contracts.
- `packages/mcp`: MCP clients, transports, tool discovery, result mapping, and URL safety.
- `packages/provider-openai`, `packages/provider-anthropic`,
  `packages/provider-gemini`, `packages/provider-mistral`: provider adapters
  mapping Anvia completion/embedding/media contracts to vendor SDKs.
- `packages/vector-chroma`, `packages/vector-lancedb`,
  `packages/vector-milvus`, `packages/vector-pgvector`,
  `packages/vector-pinecone`, `packages/vector-qdrant`,
  `packages/vector-redis`, `packages/vector-weaviate`: vector store adapters.
- `packages/embedding-transformers`: local embedding adapter.
- `packages/observability-langfuse`, `packages/observability-otel`: tracing,
  eval reporting, scoring, prompt/dataset helpers, and OpenTelemetry adapters.
- `packages/logger`: console, pino, and observer logger helpers.
- `packages/react`: React hooks and transports for chat/completion UI streams.
- `packages/server`: JSONL, SSE, and UI stream response helpers.
- `packages/tool-sandbox`: Docker-backed sandbox tools. Docker integration tests
  are gated by `ANVIA_SANDBOX_DOCKER_TESTS=1`.
- `packages/tool-browser`: caller-owned Docker browser lifecycle, visible Chromium,
  semantic browser tools, and the source image used by Studio noVNC views. Docker
  integration tests are gated by `ANVIA_BROWSER_DOCKER_TESTS=1`.
- `packages/tool-studio`: local Studio runtime, HTTP routes, storage, trace
  handling, and Vite/React UI. Its build compiles both the package and UI assets.

## Testing Notes

- Unit tests use Vitest.
- React tests use `happy-dom`.
- Studio tests exercise runtime routes, persistence, trace/session behavior, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anvia-hq/anvia](https://github.com/anvia-hq/anvia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
