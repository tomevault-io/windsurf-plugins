---
trigger: always_on
description: This repository uses the Effect Typescript library.
---

# Learning more about the Effect

This repository uses the Effect Typescript library.

Before writing any Effect code, first read `node_modules/effect/AGENTS.md`
**completely**, and follow the links in the file when required.

If you need to learn more about particular Effect apis and concepts that the
guide doesn't cover, search through the source code in `node_modules/effect/src`.

# Effect Atom client boundary

Effect Atom owns client queries, mutations, shared state, and workflows.
Keep business logic in Effect: compose multi-step client workflows as atoms,
declare cross-query invalidation as reactivity keys on mutations, and keep
promise-mode dispatches at the React boundary logic-free — no `.then` chains
in components or routes.

## Project command policy

Vite+ is the unified toolchain and command authority for this repository. It wraps Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task behind the `vp` CLI; Vite+ is distinct from Vite.

Run `vp help` for available commands and `vp <command> --help` for command-specific options. Documentation is available locally in `node_modules/vite-plus/docs` and online at https://viteplus.dev/guide/.

Use these repository commands:

- Install dependencies: `vp install`.
- Full handoff gate: `vp run ready`.
- Repository static and export checks: `vp run check`.
- Static checks: `vp check`.
- Format check: `vp fmt --check`; format fixes: `vp fmt`.
- Lint only: `vp lint`; lint fixes: `vp lint --fix`.
- Tests only: `vp test`.
- Other repository tasks and package scripts: `vp run <task>`.
- Toolchain or runtime troubleshooting: run `vp env doctor` and include its output when asking for help.

Do not use `bun run`, `npm run`, `pnpm run`, or `yarn run` in this repository. Do not invoke underlying tools such as `tsc`, `vitest`, `oxlint`, or `oxfmt` directly; use the Vite+ entry points above.

# Instructions for implementation agents

This repository is designed to be implemented by a large, parallel AI-assisted project. Every
agent must preserve a common domain language, dependency direction, and durability contract.

## Required reading

Before editing code:

1. Read `README.md`.
2. Read `GLOSSARY.md` when changing domain concepts or public terminology.
3. Read `docs/TOOLCHAIN.md`.
4. Read the relevant guide, API comments, and neighboring tests for the modules in scope.
5. Read `node_modules/effect/AGENTS.md` before writing Effect code (the canonical Effect
   guidance; `.agents/skills` carries the focused task skills).
6. Read `.agents/skills/effect-development/references/cli/index.md` before creating or
   changing repository scripts.
7. Inspect neighboring package tests before introducing a new pattern.

Keep user-facing behavior in existing guides, implementation contracts beside the code, and
verification evidence in the task or PR artifacts. Explain change rationale in the pull request.
Do not commit separate specifications, planning documents, decision registers, ADRs, roadmaps,
or investigation logs to the product repository.

## Documentation

Write documentation for humans learning how the library works and how to use it.
Keep it terse: explain the mental model, how pieces fit, and essential usage.
Make code self-documenting through clear names, types, schemas, and structure;
agents can read the implementation.

- Use small diagrams and code snippets only when they clarify ownership or usage.
- Put detailed options, defaults, and API behavior in scannable reference pages.
  Link to runnable examples for complete setup.
- Keep implementation contracts in source, schemas, and API comments.
- Keep crucial caveats beside the relevant concept; link to reference details.
- Edit the page as a whole. Do not append feature inventories, change histories,
  or long defensive explanations to an otherwise focused guide.
- Use `bun add` for consumer package installation examples.
- Draw architecture flows with `FlowMap` and request sequences with `Trace` from
  `@yielded/starlight-theme/components`, in an `.mdx` page. Give each `FlowMap` a
  `title` and `description`; the shared theme owns layout, ownership colors, and theming.
- Never hardcode Effect's current version in documentation, including READMEs,
  guides, reference pages, contributor docs, and installation commands. Use plain
  `effect` without a version or release tag in install examples. Package manifests
  and the root catalog own exact versions and peer compatibility; refer to them
  instead of repeating version numbers in prose or tables.

## Non-negotiable architecture rules

1. Public asynchronous operations return `Effect` or `Stream`, not naked `Promise` values.
2. Expected failures remain typed in `E`; dependency requirements remain visible in `R`.
3. Effect `Schema` is the canonical source for persisted and transported values.
4. Every acquired resource belongs to `Scope`. The library must not create daemon fibers.
5. Security decisions fail closed. Credentials, provider claims, and caller payloads are untrusted.
6. Private credential delivery stays outside public operation results and telemetry.
7. No code may claim exactly-once external side-effect execution. Unknown commit outcomes do
   not authorize repeating credential issuance or provider exchange.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yielded-dev/auth](https://github.com/yielded-dev/auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
