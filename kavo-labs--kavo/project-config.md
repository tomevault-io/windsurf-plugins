---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Kavo is

A production-grade CRUD framework for TypeScript: define an entity once (via TypeORM, Prisma, Mongoose, or MikroORM) and get the full REST CRUD surface — filtering, sorting, pagination, nested includes, field selection, optional per-operation DTOs, transactions, and problem-details errors — behind generated NestJS routes, configurable at global → entity → operation → per-call scope.

The authoritative sources are `docs/` (architecture notes and ADRs) and the **Conventions** section below, which is normative — naming deviations are review findings. Consult the governing ADR before changing behavior it covers, rather than inventing behavior.

## Commands

```bash
pnpm install
pnpm check        # the full gate: generate (Prisma client + fixture schema) + build + typecheck + depcruise + lint + test (run before considering work done)
pnpm build        # tsc -b (project references across the workspace — src only)
pnpm typecheck    # tsc --noEmit over the root tests/ plus every package's and example's tests/ (tsconfig.tests.json)
pnpm test         # vitest run (whole monorepo)
pnpm test:coverage # the same suite under v8 coverage, failing on the thresholds in vitest.coverage.config.ts — its own CI job, deliberately not part of `check`
pnpm depcruise    # enforce package-boundary rules (.dependency-cruiser.cjs)
pnpm lint         # oxlint over packages/*, examples/*, tests/, .github/scripts/ and tools/security-testkit/
pnpm prettify     # prettier --write . (printWidth 120)
pnpm format:check # prettier --check . — the separate formatting job CI runs alongside the gate
pnpm docs:build   # vitepress build docs — a second CI gate that `check` does NOT run
pnpm docs:links   # every `docs/**.md` reference and sidebar link resolves (a third; also not in `check`)
                  # docs/ also feeds Docs7 via docs/docs.json, so pages are MDX-dialect — tests/docs-mdx.spec.ts compiles each one
```

Run a single test file or test by name:

```bash
pnpm vitest run packages/core/tests/filter-parser.spec.ts
pnpm vitest run -t "coerces JavaScript number syntax"
```

Tests live in each package's `tests/` directory (never in `src/`, so they are not shipped in `dist/`). The one exception is the repo-level `tests/` directory, for tests whose subject is the repo's own wiring rather than any package — `tests/release-workflow.spec.ts` gates `.github/workflows/publish.yml`, and `tests/check-doc-links.spec.ts` gates `scripts/check-doc-links.sh`. Put a test there only when it belongs to no package; it is type-checked by the root `tsconfig.tests.json`. The shared security conformance suite lives in `tools/security-testkit` (private, never published, imported as `kavo-security-testkit`): each adapter and surface runs it through a driver in its own `tests/` (see that directory's README). Vitest aliases `@kavo/*` to package `src/` directly (see `vitest.config.ts`), so tests exercise sources with no stale-`dist` hazard. The SWC vitest plugin is required — TypeORM entities and Nest DI need decorator metadata that esbuild cannot emit.

Because the build compiles `src` only, each package also has a `tsconfig.tests.json` (`noEmit`, `include: ["tests"]`, `paths` mirroring the vitest aliases) that `pnpm typecheck` runs. That is what makes the type-level acceptance tests in `packages/**/tests/types/*.test-d.ts` real: `vitest.config.ts` collects only `*.spec.ts`, so nothing in them ever executes — `expectTypeOf` assertions and `@ts-expect-error` directives are checked by `tsc` alone. An unused `@ts-expect-error` is itself an error, so those tests fail in both directions.

## Architecture

Nine published packages in a hub-and-spoke topology (`pnpm-workspace.yaml`, which globs in the three `examples/*` apps as well), plus one sanctioned sideways edge:

```
@kavo/nest ──▶ @kavo/core ◀── @kavo/typeorm
   │            ▲ ▲ ▲ ▲ ▲ ▲
   │            │ │ │ │ │ └─── @kavo/prisma
   │            │ │ │ │ └───── @kavo/mongoose
   │            │ │ │ └─────── @kavo/mikroorm
   │            │ │ └───────── @kavo/sse
   │            │ └─────────── @kavo/mcp     ◀─┐
   │            └───────────── @kavo/graphql ◀─┤
   └────── the one sanctioned sideways edge ───┘
          (frameworks/* → protocols/*, ADR-0016; never the reverse)
```

- **`@kavo/core`** (`packages/core`) — all contracts, the type system, and the request engine. **Zero runtime dependencies** and imports nothing (ADR-0005). It has no knowledge of TypeORM or Nest.
- **`@kavo/typeorm`** (`packages/orms/typeorm`) — implements core's `RepositoryAdapter` and feeds core's entity-metadata seam from TypeORM metadata. `typeorm` is a peer dependency.
- **`@kavo/prisma`** (`packages/orms/prisma`) — the same seams over a Prisma Client delegate, fed from Prisma's DMMF. Needs caller-declared marker classes as entity identities (ADR-0017). `@prisma/client` is a peer dependency.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kavo-labs/kavo](https://github.com/kavo-labs/kavo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
