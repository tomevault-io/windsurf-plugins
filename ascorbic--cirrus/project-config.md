---
trigger: always_on
description: This file provides guidance to agentic coding tools when working with code in this repository.
---

This file provides guidance to agentic coding tools when working with code in this repository.

## CRITICAL: Working Directory and Plan Document

**ALWAYS verify your current working directory before operating on files:**

- Repository root is `cirrus` not `packages/pds/`
- Use `pwd` or check `process.cwd()` to confirm location
- Many project files (CLAUDE.md, plans/) are at repository root
- Package-specific files are in `packages/pds/`

**ALWAYS read and update implementation plans:**

- Plans are organized in the `plans/` directory at repository root:
  - `plans/complete/` - Completed features with full documentation
  - `plans/in-progress/` - Active development work
  - `plans/todo/` - Planned future features and improvements
- **Read** relevant plan documents before starting work to understand project status and prior decisions
- **Update** plan documents when you complete features, discover important implementation details, or change priorities
- Key plan documents:
  - `plans/complete/core-pds.md` - Core PDS implementation (all completed features)
  - `plans/todo/endpoint-implementation.md` - Endpoint implementation status and priorities
  - `plans/todo/oauth-provider.md` - OAuth 2.1 implementation plan
  - `plans/todo/migration-wizard.md` - Account migration UX specification

## Repository Structure

This is a monorepo using pnpm workspaces with the following structure:

- **Root** (`cirrus`): Workspace configuration, shared tooling, plan documents
- **packages/pds**: The main PDS library (`@getcirrus/pds`)
- **packages/oauth-provider**: OAuth 2.1 Provider (`@getcirrus/oauth-provider`)
- **packages/create-pds**: CLI scaffolding tool (`create-pds`)
- **demos/pds**: Demo PDS deployment

## Commands

### Root-level commands (run from repository root):

- `pnpm build` - Build all packages
- `pnpm test` - Run tests for all packages
- `pnpm check` - Run type checking and linting for all packages
- `pnpm format` - Format code using Prettier

### Package-level commands (run within individual packages):

- `pnpm build` - Build the package using tsdown (ESM + DTS output)
- `pnpm dev` - Watch mode for development
- `pnpm test` - Run vitest tests
- `pnpm check` - Run publint and @arethetypeswrong/cli checks

## Development Workflow

- Uses **pnpm** as package manager
- **tsdown** for building TypeScript packages with ESM output and declaration files
- **vitest** for testing
- **publint** and **@arethetypeswrong/cli** for package validation
- **Prettier** for code formatting (configured to use tabs in `.prettierrc`)

## Package Architecture

Each package in `packages/` follows this structure:

- `src/index.ts` - Main entry point
- `test/` - Test files
- `dist/` - Built output (ESM + .d.ts files)
- Package exports configured for ESM-only with proper TypeScript declarations

## TypeScript Configuration

Uses strict TypeScript configuration with:

- Target: ES2022
- Module: preserve (for bundler compatibility)
- Strict mode with additional safety checks (`noUncheckedIndexedAccess`, `noImplicitOverride`)
- Library-focused settings (declaration files, declaration maps)

## PDS Package Specifics

### Testing with Cloudflare Workers

The PDS package uses **vitest 4** with `@cloudflare/vitest-pool-workers` PR build (#11632):

- Test configuration in `vitest.config.ts` using `cloudflareTest` plugin
- Pool options: `maxWorkers: 1` and `isolate: false` for Durable Object testing
- Test environment bindings configured in `.dev.vars` (not checked into git)
- Use `cloudflare:test` module for `env` and `runInDurableObject` helpers
- Use `cloudflare:workers` module for type imports like `DurableObject`, `Env`

### TypeScript Module Resolution

The PDS package TypeScript configuration:

1. **Module Resolution**: Uses `moduleResolution: "bundler"` in tsconfig.json
2. **Test Types**: `test/tsconfig.json` includes `@cloudflare/vitest-pool-workers/types` for cloudflare:test module
3. **Import Style**: Use named imports (not namespace imports) for `verbatimModuleSyntax` compatibility

### Durable Objects Architecture

- **Worker** (stateless): Routing, authentication, DID document serving
- **AccountDurableObject** (stateful): Repository operations, SQLite storage
- **RPC Pattern**: Use DO RPC methods (compatibility date >= 2024-04-03), not fetch handlers
- **RPC Types**: Return types must use `Rpc.Serializable<T>` for proper type inference
- **Error Handling**: Let errors propagate naturally, create fresh DO stubs per request
- **Initialization**: Use lazy initialization with `blockConcurrencyWhile` for storage and repo setup

### Environment Variables

Required environment variables (validated at module load using `cloudflare:workers` env import):

- `DID` - The account's DID (did:web:...) - validated with `isDid()`
- `HANDLE` - The account's handle - validated with `isHandle()`
- `PDS_HOSTNAME` - Public hostname
- `AUTH_TOKEN` - Bearer token for write operations (simple auth)
- `SIGNING_KEY` - Private key for signing commits
- `SIGNING_KEY_PUBLIC` - Public key multibase for DID document

**Optional (for session-based auth):**

- `JWT_SECRET` - Secret for signing session JWTs
- `PASSWORD_HASH` - Bcrypt hash of account password

**Optional (for blob storage):**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ascorbic/cirrus](https://github.com/ascorbic/cirrus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
