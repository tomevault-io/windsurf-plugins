---
trigger: always_on
description: This file contains instructions for AI coding agents operating in this repository.
---

# Agents

This file contains instructions for AI coding agents operating in this repository.

The project's GitHub URL is https://github.com/skorokithakis/stavrobot/.

## Architecture reference

`ARCHITECTURE.md` is the authoritative reference for how the system works: containers,
message flow, database schema, security model, plugin system, and more. Read it before
exploring the repo — it will answer most structural questions faster than grepping. After
any change that affects architecture (new containers, endpoints, tools, database tables,
security boundaries, message flow, etc.), update `ARCHITECTURE.md` to reflect the new
state.

## Project overview

Stavrobot is a TypeScript HTTP server that wraps an LLM-powered agent (Anthropic Claude
by default) with access to a PostgreSQL database via a SQL execution tool. It exposes a
single `POST /chat` endpoint. The project is containerized with Docker and docker-compose.

## Build, run, and test commands

```bash
# Install dependencies
npm install

# Build (compile TypeScript to dist/)
npm run build

# Run the compiled server
npm start

# Run in dev mode (tsx, no compile step)
npm run dev

# Type-check without emitting (useful for CI or quick validation)
npx tsc --noEmit

# Run tests
npm test

# Docker
docker compose up --build
```

Tests use vitest. Run `npm test` after changes that touch request handling, auth,
or database logic.

There is no linter or formatter configured. If one is added, use Biome (single tool for
both linting and formatting with good TypeScript ESM support).

## Code style: TypeScript

### Module system

- The project uses ESM (`"type": "module"` in package.json, `"module": "NodeNext"` in
  tsconfig).
- All local imports must use the `.js` extension (e.g., `import { loadConfig } from
  "./config.js"`), even though the source files are `.ts`. This is required by NodeNext
  module resolution.

### Imports

- Use `import type` for type-only imports (e.g., `import type { Config } from
  "./config.js"`). This is enforced by strict mode and tree-shaking.
- Order: third-party modules first, then local modules (with `./` prefix).
- Named exports only. No default exports.

### Typing

- `strict: true` is enabled in tsconfig. All code must satisfy strict type checking.
- All functions must have explicit return type annotations, including `Promise<void>`.
- Use interfaces for object shapes (e.g., `Config`, `PostgresConfig`).
- Use `unknown` instead of `any` for untyped data, then narrow with type guards or
  assertions. The only exception is when a library API forces `any`.
- Type assertions should be used sparingly. When necessary, cast through `unknown` first
  (e.g., `TOML.parse(content) as unknown as Config`).

### Naming

- `camelCase` for variables, functions, and parameters.
- `PascalCase` for interfaces and type aliases.
- No abbreviations. Prefer `message` over `msg`, `response` over `res`, `request` over
  `req`.

### Functions and structure

- Use standalone `async` functions, not classes, for application logic.
- The `Agent` class from the library is the only class used directly.
- Keep functions small and focused. Each file in `src/` has a clear responsibility:
  `config.ts` (loading config), `database.ts` (Postgres operations), `agent.ts`
  (LLM agent setup and prompting), `index.ts` (HTTP server and entry point).

### Logging

- Add `console.log` statements to trace code flow at a high level. Not every line, but
  enough to follow what's happening in the logs: tool invocations, external API calls,
  key decision points, and results (or at least their size/shape).
- Use the `[stavrobot]` prefix for all log lines (e.g.,
  `console.log("[stavrobot] web_search called:", query)`).
- Use `console.error` for errors and `console.warn` for warnings.

### Error handling

- Let exceptions propagate. Do not add defensive try/catch blocks unless the function
  is the boundary where the error must be handled (e.g., the HTTP request handler in
  `index.ts`).
- At HTTP boundaries, catch errors and return structured JSON error responses with
  appropriate status codes.
- Use `error instanceof Error ? error.message : String(error)` when converting unknown
  caught values to strings.

### Comments

- Comments explain "why", not "what". Do not add comments that just describe what the
  next few lines do.
- Comments should be full sentences, properly capitalized, ending with a full stop.
- Currently the codebase has almost no comments because the code is straightforward. Do
  not add unnecessary comments.

### Formatting

- Use double quotes for strings.
- Use `const` by default, `let` only when reassignment is needed. Never use `var`.
- Trailing commas in multiline constructs.
- Semicolons at end of statements.
- 2-space indentation.

## Configuration

- Runtime config is loaded from `config.toml` (or path in `CONFIG_PATH` env var).
- `config.toml` is gitignored. `config.example.toml` is the template.
- Do not commit secrets or API keys.

## Docker

- Multi-stage Dockerfile: build stage compiles TypeScript, production stage copies only
  compiled JS and production dependencies.
- docker-compose runs Postgres 17 and the app, with a health check on Postgres before
  the app starts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skorokithakis/stavrobot](https://github.com/skorokithakis/stavrobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
