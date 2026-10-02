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

This repository consumes APIs through Effect Atom clients (`@effect/atom-react`).
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
   Before changing runtime, storage, or platform packages, also read the authoritative
   [runtime model](docs/src/content/docs/concepts/runtime-model.md).
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

Documentation is for humans learning the library. Guides must be terse and explain
concepts succinctly: what a feature does, how it fits, and how to use it.

- Lead with the mental model and ownership boundaries. Use small architecture or
  flow diagrams and only the code snippets essential to understanding and usage.
- Put detailed options, defaults, and API behavior in scannable reference pages.
  Link to runnable examples for complete setup.
- Keep implementation contracts in source, schemas, and API comments. LLMs can
  read the code; do not turn user guides into agent context or implementation audits.
- Keep crucial caveats beside the relevant concept; link to reference details.
- Edit the page as a whole. Do not append feature inventories, change histories,
  or long defensive explanations to an otherwise focused guide.

## Non-negotiable architecture rules

1. Public asynchronous operations return `Effect` or `Stream`, not naked `Promise` values.
2. Expected failures remain typed in `E`; dependency requirements remain visible in `R`.
3. Effect `Schema` is the canonical source for persisted, transported, tool, and structured model
   values.
4. Every acquired resource belongs to `Scope`. The engine must not create daemon fibers.
5. Use the pinned Effect v4 AI primitives directly. Do not introduce framework-owned copies of
   Effect AI `Tool`, `Toolkit`, `LanguageModel`, `Prompt`, `Response`, or `Model`.
6. Provider SDK values never become canonical thread records. Effect AI values may be used
   by the interpreter, but durable records remain explicit, versioned Schemas.
7. The canonical log is append-only. Projections and checkpoints are disposable derivatives.
8. No code may claim exactly-once external side-effect execution.
9. An unresolved ordinary tool call is never automatically replayed after ownership loss.
10. Tool/model/subagent concurrency is bounded and deterministic at commit time. Tool batches use
    Effect structured concurrency and Semaphore permits rather than a separate Promise scheduler.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yielded-dev/agent](https://github.com/yielded-dev/agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
