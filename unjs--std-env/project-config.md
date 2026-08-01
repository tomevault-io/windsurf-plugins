---
trigger: always_on
description: `std-env` — runtime-agnostic JS utility library for detecting environments, runtimes, CI providers, and AI coding agents. Published as `std-env` on npm under `unjs/std-env`.
---

# Agents

## Project Overview

`std-env` — runtime-agnostic JS utility library for detecting environments, runtimes, CI providers, and AI coding agents. Published as `std-env` on npm under `unjs/std-env`.

## Maintenance

- **`AGENTS.md`** — Keep updated with technical details, architecture, and conventions for AI agents working on this project
- **`README.md`** — Keep updated with user-facing documentation (usage, installation, exported APIs, examples)

When adding new features or changing behavior, update both files accordingly.

**After finishing any code updates, always run `pnpm lint:fix` then `pnpm test` to auto-fix lint/formatting issues and verify everything passes.**

- ESM-only (`"type": "module"`, single `.mjs` output)
- Linted with `oxlint` + `oxfmt`, typechecked with `tsgo`
- Tested with `vitest`
- Built with `obuild`

## Scripts

| Script               | Description                                              |
| -------------------- | -------------------------------------------------------- |
| `pnpm run build`     | Build with obuild (`dist/index.mjs`, `dist/index.d.mts`) |
| `pnpm run dev`       | Start vitest in watch mode                               |
| `pnpm run test`      | Lint + typecheck + vitest with coverage                  |
| `pnpm run lint`      | Run oxlint and oxfmt                                     |
| `pnpm run lint:fix`  | Auto-fix lint/format issues                              |
| `pnpm run typecheck` | Run tsgo --noEmit                                        |

## Source Structure

All source lives in `src/`, single entry point at `src/index.ts` which re-exports everything.

| File               | Purpose                                                                                                         |
| ------------------ | --------------------------------------------------------------------------------------------------------------- |
| `src/index.ts`     | Barrel re-export of all modules                                                                                 |
| `src/agents.ts`    | AI coding agent detection (`detectAgent`, `agentInfo`, `agent`, `isAgent`)                                      |
| `src/providers.ts` | CI/CD provider detection (`detectProvider`, `providerInfo`, `provider`)                                         |
| `src/runtimes.ts`  | JS runtime detection (`runtime`, `runtimeInfo`, `isNode`, `isBun`, `isDeno`, etc.)                              |
| `src/flags.ts`     | Environment flags (`isCI`, `isDebug`, `isTest`, `isProduction`, `isDevelopment`, `isMinimal`, `platform`, etc.) |
| `src/env.ts`       | Universal `env` proxy + `nodeENV` constant                                                                      |

## Benchmarks

Micro-benchmarks live in `test/bench/` and use [mitata](https://github.com/evanwashere/mitata). Run with:

```bash
node test/bench/agents.ts
```

## Essential Rules

- **No global `process` usage.** Never use `globalThis.process` or bare `process` directly. Always import `env` from `src/env.ts` and use it to access environment variables. This ensures runtime-agnostic behavior across all environments.

## Detection Patterns

All detection modules follow the same pattern:

1. Define a `Name` type union of known values (extensible via `string & {}`)
2. Define an internal tuple array mapping names to env vars (with optional metadata)
3. Implement a `detect*()` function that iterates tuples checking env
4. Export a lazy-evaluated singleton (`*Info`) and convenience accessors

### Agent Detection (`src/agents.ts`)

- **Priority**: `AI_AGENT` env var (generic override) → ordered tuple scan
- Internal types:
  - `EnvCheck = string | ((env: Record<string, string | undefined>) => boolean)`
  - `InternalAgent = [agentName: AgentName, envChecks: EnvCheck[]]`
- Each agent maps to an array of `EnvCheck` items; the first matching check wins
- When `EnvCheck` is a string, the env var must be truthy; when it's a function, it receives the full env object and returns a boolean
- `envMatcher(envKey, regex, opts?)` helper creates regex-based env var matchers (e.g., `envMatcher("TERM_PROGRAM", /kiro/)`)
  - `opts.noTTY: true` makes the matcher only match in a non-interactive context (when `process.stdout.isTTY` is falsy). Use for detections keyed off generic terminal env vars that an IDE sets in its integrated terminal too — e.g. `kiro` (`TERM_PROGRAM=kiro`), where a TTY means a human is at the IDE terminal, not the agent CLI (see #185)
- IDEs (cursor, devin, kiro) are checked last so agents running inside them are detected first
- Exports: `detectAgent()`, `agentInfo` (singleton), `agent` (name shorthand), `isAgent` (boolean)

### Provider Detection (`src/providers.ts`)

- 50+ CI/CD providers detected via env vars
- Some providers have `{ ci: false }` metadata (e.g., Vercel, CodeSandbox — deployment platforms, not CI)
- Some providers have `{ ci: true }` explicitly (e.g., AWS Amplify, Cloudflare Pages/Workers)
- Special case: StackBlitz/WebContainer detected via `SHELL === "/bin/jsh"` + `process.versions.webcontainer`

### Build & Tree-shaking (`build.config.ts`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unjs/std-env](https://github.com/unjs/std-env) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
