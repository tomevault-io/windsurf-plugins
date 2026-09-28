---
trigger: always_on
description: Contains the unscoped `cf` public npm CLI and its code generator. Shared code lives in
---

# cf repo — AGENTS.md

Contains the unscoped `cf` public npm CLI and its code generator. Shared code lives in
`@cloudflare/forge`, vendored as a tarball in `vendor/` until it is on npm.

## Cross-platform support

`cf` supports Windows, Linux, and macOS. When designing or implementing every
feature, account for differences between all three operating systems rather
than assuming POSIX behaviour. Pay particular attention to process spawning,
signals and exit codes, shells and argument quoting, filesystem paths and
permissions, environment variables, and terminal behaviour. Use
platform-specific implementations and tests where their semantics differ.

## Layout

```
cf/
├── packages/cli/       # cf — CLI + generator
│   ├── src/
│   │   ├── index.ts            # yargs entry; registers lazy-loaded generated
│   │   │                       # commands + hand-written commands
│   │   ├── dev.ts              # tsx entry wrapper (maps CliExit → process.exit);
│   │   │                       # NOT the `cf dev` command
│   │   ├── version.ts          # VERSION resolver from bundled package JSON
│   │   ├── config.ts           # public config types / defineConfig entry
│   │   ├── sdk/                # tracked generated SDK; rewritten by generation
│   │   ├── commands/
│   │   │   ├── _generated/     # tracked generated source; rewritten by pnpm generate
│   │   │   ├── hand-written.ts # central root/leaf/override/subgroup registry
│   │   │   ├── auth/           # hand-written: login, logout, whoami, profiles
│   │   │   ├── login/          # hidden `cf login` alias for `cf auth login`
│   │   │   ├── cli/            # hand-written: search and telemetry settings
│   │   │   ├── telemetry/      # implementation of `cf cli telemetry`
│   │   │   ├── completions/    # hand-written: `cf complete` (@bomb.sh/tab)
│   │   │   ├── dev/            # hand-written: `cf dev` (impl discovery + spawn)
│   │   │   ├── build/          # hand-written: direct framework/delegate build
│   │   │   ├── deploy/         # hand-written: `cf deploy` (Build Output → upload)
│   │   │   ├── migrate/        # hand-written: Wrangler-to-cf config migration
│   │   │   ├── previews/       # hand-written: `cf previews deploy`
│   │   │   ├── init/           # hand-written: `cf init` / `cf init workers`
│   │   │   │                   # (hello-world scaffold or autoconfig setup)
│   │   │   ├── containers/     # hand-written: local image build/push leaves
│   │   │   ├── tunnels/        # hand-written cloudflared-backed leaves
│   │   │   ├── workers/check/          # hand-written local startup profiling leaf
│   │   │   ├── workers/triggers/       # hand-written `cf workers triggers` subgroup
│   │   │   ├── workers/versions/create/# hand-written Build Output leaf override
│   │   │   ├── d1/migrations/  # hand-written: `cf d1 migrations` (spliced into
│   │   │   │                   # the generated d1 tree — see hand-written-overrides.ts)
│   │   │   ├── schema.ts       # hand-written: dump JSON-schema for an op
│   │   │   └── tools.ts        # hand-written: list/print MCP tool definitions
│   │   ├── lib/                # see "Library Map" below
│   │   └── __tests__/          # vitest + MSW harness (unit + command tests)
│   ├── generator/
│   │   ├── index.ts            # entry transformer; emits files via forge.emit()
│   │   ├── generator.ts        # per-command emitter (handler body, args)
│   │   ├── metadata.ts         # _meta/ JSON sidecars + schema dump
│   │   ├── arg-classification.ts # path/query/body/header bucketing
│   │   ├── arg-derivation.ts   # derived/computed arg handling
│   │   ├── codegen/            # TS code-emission helpers
│   │   └── emit/               # per-file emit helpers
│   ├── e2e/                    # 114 JSON product fixtures +
│   │                           # generated bash runner (run-e2e.sh) hitting a
│   │                           # real Cloudflare account
│   ├── bin/cf                  # binary entry — loads lightweight delegation,
│   │                           # then imports dist/index.mjs when staying local
│   ├── dist/                   # gitignored, written by tsdown (chunked ESM)
│   ├── generate.ts             # OpenAPI/SDK sync → initFromOpenApi → transform/finalize
│   ├── vitest.config.mts       # in-package test harness config
│   └── tsdown.config.ts        # ESM bundle config; package JSON supplies
│                               # the version; `define` can inject the prerelease label
├── packages/wrangler-tests/    # imported wrangler test corpus, aliased onto
│                               # ../cli/src/index.ts (MSW-mocked)
├── fixtures/                   # cf dev discovery fixtures (vite-plugin +
│                               # wrangler-bundler projects)
├── usecases/                   # per-command usage-scenario YAML catalogue
├── vendor/                     # Vendored tarballs: forge + SDK transformer
├── patches/                    # pnpm patches: release publishing + Container image
│                               # cleanup ownership
├── scripts/sync-forge.ts       # re-vendor pipeline (FORGE_REPO=...)
├── .github/workflows/          # CI, changesets publish, prerelease, benchmark, review

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/cf](https://github.com/cloudflare/cf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
