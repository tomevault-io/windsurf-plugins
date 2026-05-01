---
trigger: always_on
description: Welcome. This is a contract‑first, agent‑friendly data layer.
---

# Agents — Prisma Next

Welcome. This is a contract‑first, agent‑friendly data layer.

## Start Here

- [Docs Index](docs/README.md) — How the docs are organized and what to read next
- [Architecture Overview](docs/Architecture%20Overview.md) — High-level design principles
- [Testing Guide](docs/Testing%20Guide.md) — Philosophy, patterns, and commands
- [Rules Index](.cursor/rules/README.md) — All Cursor rules organized by topic
- [ADRs](docs/architecture%20docs/adrs/) — Architecture Decision Records

### Modular Onboarding

- [Getting Started](docs/onboarding/Getting-Started.md) — Build, test, and run demo
- [Repo Map & Layering](docs/onboarding/Repo-Map-and-Layering.md) — Package organization and import rules
- [Conventions](docs/onboarding/Conventions.md) — TypeScript, tooling, and code style
- [Testing](docs/onboarding/Testing.md) — Test commands, patterns, and organization
- [Common Tasks Playbook](docs/onboarding/Common-Tasks-Playbook.md) — Add operation, split monolith, fix import

## Project Overview

**Prisma Next** is a contract-first data access layer:

- **Contract-first**: Emit `contract.json` + `contract.d.ts` — no executable runtime code generation
- **Composable DSL**: Type-safe query builder (`sql().from(...).select(...)`)
- **Machine-readable**: Structured artifacts that agents can understand and manipulate
- **Runtime verification**: Contract hashes and guardrails ensure safety before execution

## Golden Rules

- **Node.js version**: Use the shell's Node. Do not run nvm/fnm or any version switcher. Source of truth is root `package.json` `engines.node`. If the shell's version is wrong or commands fail, report that the shell is misconfigured and the user should configure their environment to satisfy `engines.node`.
- Use `pnpm` not `npm`!
- The source of truth for the required Node version is the root `package.json` `engines.node` field.
- Never use `npx`
- If the shell's `node -v` does not satisfy that, or Node/pnpm commands fail due to version, report:
  "Your shell is misconfigured for this repo. Configure your environment so that `node`
  resolves to a version that satisfies the root package.json engines.node (e.g. set your
  default Node in your version manager, or use Volta)."
- We use turborepo to build, generally. `pnpm build` scripts are available in each package which delegate to turborepo
- If you change exported types in a workspace package consumed by other packages/examples, run that package's `pnpm build` to refresh `dist/*.d.mts` declarations before validating downstream TypeScript usage.
- When you want to typecheck, use the local `pnpm typecheck` scripts instead of writing `tsc` commands from scratch
- When you want to test, use the local `pnpm test` scripts. For coverage, use `pnpm test:coverage` (all packages) or `pnpm coverage:packages` (packages only, excludes examples)
- Use arktype instead of zod
- Never add file extensions to imports in TypeScript
- Don't add comments if avoidable, prefer code which expresses its intent
- Don't add exports for backwards compatibility unless requested to do so
- Always write tests before creating or modifying implementation
- Do not reexport things from one file in another, except in the `exports/` folders
- Don't branch on target; use adapters: `.cursor/rules/no-target-branches.mdc`
- Keep tests concise; omit "should": `.cursor/rules/omit-should-in-tests.mdc`
- Keep docs current (READMEs, rules, links): `.cursor/rules/doc-maintenance.mdc`
- Prefer links to canonical docs over long comments

## Typesafety rules
- Never use `any` type
- Never use `@ts-expect-error` outside of negative type tests
- Never use `@ts-nocheck`
- Never suppress biome lints
- Try to minimize type casts. Instead, prefer explicit types that would make type casts unnecessary. If type cast is unavoidable, try to minimize its scope — never cast
  entire object/class when casting one property would suffice.
- `as unknown as SomeOtherType` type cast should be used only as a last resort and should always be accompanied by a comment explaining why it is necessary.

## Common Commands

```bash
pnpm build                 # Build via Turbo
pnpm test:packages         # Run package tests
pnpm test:e2e              # Run e2e tests
pnpm test:integration      # Run integration tests
pnpm test:all              # Run all tests
pnpm coverage:packages     # Coverage (packages only)
pnpm lint:deps             # Validate layering/imports
```

## Core Concepts

### Contract Flow

1. **Authoring**: Write `schema.psl` or use TypeScript builders → canonicalized Contract IR
2. **Emission**: Emitter validates and generates `contract.json` + `contract.d.ts`
3. **Validation**: `validateContract<Contract>(json)` validates structure and returns typed contract
4. **Usage**: DSL functions (`sql()`, `schema()`) accept contract and propagate types

### Key Patterns

- **Type Parameter Pattern**: JSON imports lose literal types. Use `.d.ts` for precise types, `validateContract()` for runtime validation
- **ExecutionContext**: Encapsulates contract, codecs, operations, and types. Pass to `schema()`, `sql()`, `orm()`
- **Interface-Based Design**: Export interfaces and factory functions, not classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
