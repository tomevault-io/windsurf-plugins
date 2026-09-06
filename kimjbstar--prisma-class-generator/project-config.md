---
trigger: always_on
description: Guidance for AI coding assistants (and human contributors) working in this repo.
---

# CLAUDE.md

Guidance for AI coding assistants (and human contributors) working in this repo.
For what this package *does* and how to *use* it, see [README.md](./README.md) — this file
is about working on the generator itself. For the PR mechanics (checklist, what tends to get
merged quickly), see [CONTRIBUTING.md](./CONTRIBUTING.md).

## What this is

A Prisma generator (`prisma-class-generator`) that reads a schema's DMMF and emits one
TypeScript class per model/composite-type, decorated for `@nestjs/swagger` and/or
`@nestjs/graphql`. It runs as a subprocess Prisma's CLI spawns via the JSON-RPC generator
protocol (`@prisma/generator-helper`'s `generatorHandler`), not as a library the user imports.

## Architecture

```
src/
  index.ts          generatorHandler entry point (what Prisma's CLI actually spawns)
  bin.ts             CLI entry point (the "prisma-class-generator" bin)
  generator.ts       PrismaClassGenerator — orchestrates one full run()
  convertor.ts       PrismaConvertor — DMMF.Field -> FieldComponent/ClassComponent
  components/        Class/Field/Decorator/Import/File components (string-template assembly)
  templates/         `#!{PLACEHOLDER}` string templates the components fill in
```

This is deliberately a string-template pipeline, not an AST transform. It's simple to reason
about at this size — don't reach for ts-morph or similar without a concrete reason.

Runtime `dependencies` are deliberately kept to two (`@prisma/generator-helper`, `prettier`) —
everything a user installs to run this generator. `@prisma/internals` is a **devDependency on
purpose**: the specs use its `getDMMF`, but it installs 28MB and `prisma` itself doesn't depend
on it (so it never dedupes), and the only two things the generator needed from it —
`parseEnvValue` and `logger.info` — are transcribed into `util.ts`. Before adding any runtime
import, check whether it's really worth what it costs a user; `npm run verify:packed` is what
catches a runtime import that only resolves because this repo has everything installed.

`util.ts`'s `toSnakeCase` used to be `change-case`'s
`snakeCase`, inlined once that package's only remaining pull was one function and its 5.x line
went ESM-only; its output is a **compatibility contract**, not a style choice — it produces both
the generated filename and the import path other generated files reach it by, so changing it
renames every existing user's generated files. `util.spec.ts` pins the cases that matter.

`PrismaClassGenerator` and `PrismaConvertor` are singletons (`static instance` /
`getInstance()`) — fine for a CLI that runs once per process. `FileComponent`, however, takes
`clientImportPath`/`useGraphQL`/`prettierOptions` as explicit constructor params rather than
reaching for the singleton — keep it that way; it's what makes `FileComponent` testable without
seeding global state in every spec file.

## Commands

```
npm run build          # rm -rf dist && tsc — always clean-builds, never leaves stale output
npm test                # jest
npm run format:check    # what CI enforces — `npm run lint` can't catch formatting, because
                         # eslint-config-prettier *disables* those rules rather than checking them
npm run lint:package    # publint against the packed tarball. `--pack npm` is not optional:
                         # publint mis-parses Yarn Classic's tarballs (bare directory entries)
                         # and reports every entry point as missing from `files`
npm run verify:packed   # pack, install the tarball into an empty project, run a real
                         # `prisma generate` through it by provider name — the only check that
                         # sees a bad files/bin field or a runtime import of a devDependency
npm run generate:*      # regenerate a fixture under prisma/*.prisma (postgresql, mysql,
                         # mongodb, mssql, sqlite, cockroachdb) — useful for manually eyeballing
                         # output, but note prisma-client-js's auto-install can add
                         # `@prisma/client` back into package.json as a side effect; that's
                         # intentionally a devDependency, revert it if it reappears in
                         # `dependencies`.
```

## Testing approach

- `src/**/*.spec.ts` (excluded from `tsc` build via tsconfig, picked up by `jest.config.js`).
- `src/fixtures.spec.ts` is the important one: it runs the *actual* `prisma/*.prisma` files
  (one per supported database) through the real pipeline and snapshots the full output of every
  generated file with `toMatchSnapshot()`. This is the closest thing to a golden/end-to-end test
  in this repo — fragment-level `toContain()` assertions elsewhere are useful for pinpointing
  *why* something changed, but the snapshot is what proves the whole pipeline still produces
  correct code for a real schema.
- When you intentionally change generated output, run `npx jest -u` and **read the diff** before
  committing the updated snapshot — the point of the golden test is that a snapshot diff is a
  decision point, not a checkbox.
- `PrismaConvertor`/`FileComponent` can be tested directly by `new`-ing them (bypass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kimjbstar/prisma-class-generator](https://github.com/kimjbstar/prisma-class-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
