---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## GitHub Actions: SHA Pinning (Mandatory)

All GitHub Actions MUST use commit SHA references, never version tags or branch references. This is a supply chain security requirement to prevent tag-based attacks and ensure reproducible CI/CD pipelines.

### Correct — SHA-pinned with version tag comment

```yaml
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
- uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8 # v4.0.2
```

Note: npm is included with Node.js, so no additional setup action is required.

### Wrong — tag or branch reference (not allowed)

```yaml
# Never use tags
- uses: actions/checkout@v4
- uses: actions/setup-node@v4

# Never use branches
- uses: actions/checkout@main
```

**How to find the SHA:** Use GitHub's action lookup tool or check the action's release page for the commit SHA of your desired version. Always include the version tag as a comment for readability.

---

## Monorepo Structure

Three npm workspaces in `packages/` (defined via `"workspaces": ["packages/*"]` in root `package.json`):

- **`@mctx-ai/mcp`** (`packages/server/`) — Core framework. Zero runtime dependencies. Exports `createServer`, `T`, `conversation`, `log`, `buildInputSchema`, `getLogBuffer`, `clearLogBuffer`. Type exports include `ModelContext` (`{ userId?: string }`). Build is a simple `cp src/*.js src/*.d.ts dist/` (no transpilation).
- **`@mctx-ai/dev`** (`packages/dev/`) — Dev server with hot reload, request logging, log surfacing (handler log entries printed to dev console), and sampling stub (`/_mctx/sampling` endpoint returns error in dev mode). Peer-depends on `@mctx-ai/mcp`. Uses Node.js built-in test runner (`node --test`), not Vitest. Lint is a stub (`echo 'Linting not configured yet'`).
- **`create-mctx-server`** (`packages/create-mctx-server/`) — CLI scaffolding tool (`npm create mctx-server <name>`). Generates a new project with `@mctx-ai/mcp` + `@mctx-ai/dev` + `esbuild` configured.

Root commands affect all workspaces. Use `--workspace` flag for package-specific operations.

**Requires:** Node >=22.0.0, npm >=10.8.0 (enforced in root `package.json` `engines` field).

**`.npmrc`:** `save-exact=true` — all dependencies installed with exact versions (no `^` or `~` ranges).

---

## Development Commands

### Root (all packages)

```bash
npm run build          # Build all packages
npm test              # Run all tests
npm run lint          # Lint all packages
npm run format        # Format with Prettier
npm run format:check  # Check formatting without modifying
```

### Server Package

```bash
# Testing
npm run test --workspace=@mctx-ai/mcp
npm run test:coverage --workspace=@mctx-ai/mcp  # V8 coverage, 80% thresholds
npx vitest run test/uri.test.js                        # Single test file (from packages/server/)
npx vitest run -t "test name"                          # Specific test by name

# Code quality
npm run lint --workspace=@mctx-ai/mcp
npm run lint:fix --workspace=@mctx-ai/mcp
npm run typecheck --workspace=@mctx-ai/mcp  # tsc --noEmit
```

---

## Code Conventions

### Language and Typing

- **JavaScript with ESM** — No TypeScript source files. All packages use `"type": "module"`.
- **Type definitions** — Hand-written `.d.ts` files (see `packages/server/src/index.d.ts`, 770+ lines).
- **JSDoc** — Inline documentation in JS source, exported types in `.d.ts`.

### Naming Conventions

- **Functions and variables:** `camelCase`
- **Constants:** `UPPER_SNAKE_CASE`
- **Types (in `.d.ts`):** `PascalCase`
- **Unused parameters:** `_` prefix (e.g., `function handler(_req, res)`)

### Linting

- **ESLint 9** with flat config (`eslint.config.js`)
- **Key rule:** `no-unused-vars` with `argsIgnorePattern: "^_"` — prefix unused params with underscore

### Formatting

- **Prettier 3** for `.js`, `.json`, `.md` files (`printWidth: 100` in `.prettierrc.json`)
- Run `npm run format` before committing

---

## Architecture Patterns

### Handler Descriptor Pattern

Functions carry metadata as properties:

```javascript
function greet(mctx, req, res) {
  res.send(`Hello, ${req.name}!`);
}
greet.description = "Greet someone by name";
greet.input = { name: T.string({ required: true }) };

server.tool("greet", greet);
```

Handler functions receive three parameters: `(mctx, req, res)` for all handler types (tools, resources, and prompts).

- `mctx` — model context: `{ userId?: string }`
- `req` — validated input fields accessed directly (`req.name`, `req.query`, etc.). For static resources, `req` is `{}`. For URI template resources, `req` contains the extracted template parameters.
- `res` — output port: `{ send(result), progress(current, total?), ask(prompt) }`

### Handler Types

1. **Tools** — Sync or async functions. Receive `(mctx, req, res)`. Call `res.send(result)` to return the result. Call `res.progress(current, total?)` for progress reporting. Call `res.ask(prompt)` for LLM sampling (`null` if client does not support sampling).
2. **Resources** — Static URIs or URI templates with `{param}` placeholders. Params extracted via RFC 6570 Level 1. All resource handlers receive `(mctx, req, res)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mctx-ai/mcp](https://github.com/mctx-ai/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
