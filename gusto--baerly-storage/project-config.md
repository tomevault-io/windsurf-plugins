---
trigger: always_on
description: Guidance for AI coding agents working in this repo. Keep this file lean —
---


# CLAUDE.md

Guidance for AI coding agents working in this repo. Keep this file lean —
only content that **cannot be inferred from the code** belongs here.

## What this is

**baerly-storage** is a vendorless document database that runs over
any S3-compatible storage API. The data lives in your bucket; the
protocol kernel is small enough that an LLM can use the public API
zero-shot from the `.d.ts` files alone. Theoretical foundations live
in [docs/](docs/).

**Current state:** Open source under Apache-2.0, published publicly as
`@gusto/baerly-storage` (and the `@gusto/create-baerly-storage`
scaffolder) on npm (npmjs.com) with `publishConfig.access: "public"`.
See
[`docs/contributing/publishing.md`](docs/contributing/publishing.md) for
the publish workflow.

The protocol kernel and HTTP server are landed. Day-1 templates ship
for Cloudflare Workers and self-hosted Node; both are first-class.
AWS Lambda / Bun / Deno / Fly are an adapter package away.

## Toolchain

- **Package manager:** pnpm (`packageManager: pnpm@11.1.2`).
- **Test runner:** vitest (`vitest run`). Tests import from `"vitest"`.
- **Type checker:** TypeScript 7 / `tsgo` (`@typescript/native-preview`).
- **Formatter:** oxfmt.
- **Linter:** oxlint.
- **Bundler:** rolldown (`rolldown.config.ts`).

Don't introduce alternate tooling without justification.

## Verification

Under Claude Code, `vitest` runs use the compact `minimal` reporter —
vitest 4.1 auto-detects AI-agent environments, and the repo config
(`vitest.config.ts`) additionally pins this behavior when
`CLAUDECODE=1` is set so it isn't silently broken by detection
changes. Failures still print in full. Override with
`--reporter=dot` for long suites (`test:randomize`,
`test:fuzz-maintenance`) when progress signal matters more than
compactness, or `--reporter=default` to force the full reporter.
`pnpm verify` / `pnpm test` is what humans run before pushing. `pnpm
verify` is split into `verify:code` (the whole chain minus markdown) and
`verify:docs` (markdown/mermaid validation); `verify` runs both. The
lefthook pre-commit hook runs them by glob — `verify:code` when a
`.ts`/`.tsx` file is staged, `verify:docs` when a `.md` file is staged
(so a docs-only commit is now validated locally too, and a code-only
commit skips the markdown checks it can't affect) — plus a scoped
`pnpm bundle-sizes` (see below), but NOT the full `pnpm test` suite.
`pnpm verify:agent` / `pnpm test:agent` are explicit compact-output
variants for environments where the env var isn't propagated.

> **Agents: don't pipe `verify:agent` / `test:agent` through `| tail -N` or `| head -N`.** Both scripts are already compact — one finding per line, with full detail preserved on failures. Piping to `tail`/`head` removes the lines you need; if the first run prints nothing useful, the _output is empty because the gate passed_, not because the tail was wrong. Same applies to `pnpm bundle-sizes`.

| Command                                                                 | What it catches                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Runtime                              | Clean on `main`?                                                                                                                                                      |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pnpm verify`                                                           | typecheck (`tsgo --noEmit`) + `verify:examples` + lint (`oxlint`) + `format:check` (`oxfmt --check .`, whole-repo) + `lint-format-coverage` (ownership guard) + `verify:docs` (markdown validation) + `check-spec-drift` + `check-version-matrix`                                                                                                                                                                                                                                                                      | ~seconds                             | ✅ — non-zero exit _is_ your regression                                                                                                                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gusto/baerly-storage](https://github.com/Gusto/baerly-storage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
