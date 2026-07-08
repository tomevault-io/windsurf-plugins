---
trigger: always_on
description: This repo defines a `.local` + `.docs` workflow for human-agent work.
---

# Lotus Agents - Repository Contract

## What This Repository Is

This repo defines a `.local` + `.docs` workflow for human-agent work.

The current product shape is:

- `README.md` is the main human entrypoint
- `lotus-local/` is the local-first installable skill package root
- `lotus-linear/` is the Linear-backed installable skill package root
- `.codex-plugin/plugin.json` bundles the skill collection for native Codex
  plugin installation

## What This Repository Is Not

Do not treat this repo as a consumer repository by default.

- root `.local/` is disposable private state, not the product contract
- do not keep `.local/examples` fixtures in this repository; remove them during
  cleanup unless a human explicitly asks to preserve a specific local example
- root `.docs/` may not exist
- do not bootstrap consumer `.local/` or `.docs/` in this repo unless a human
  explicitly asks

## Working Rules In This Repo

When changing the contract or adoption story:

1. keep `README.md` aligned with the relevant skills under `lotus-local/` and
   `lotus-linear/`
2. keep the model centered on `.local/AGENTS.md` and `.docs/AGENTS.md`
3. keep `.local/` private-first and `.docs/` required in the workflow but
   optional to commit
4. do not reintroduce `AGENTS_TO_COPY.md`, `AGENTS_ISSUE_FLOW.md`, `docs/`,
   `.local/context.md`, `.local/questions/`, or `.local/runs/`
5. do not reintroduce setup scripts, path configuration, or custom layout
   support unless a human explicitly asks
6. keep skills optional, task-focused, and copy-paste friendly
7. keep reusable templates in the relevant skill assets or in consumer
   `.docs/templates/`, not in one legacy copy artifact
8. never use live `.local/` session state, auth files, logs, databases, plugin
   caches, or sandbox files as examples

## Code Style

- Name files in camelCase by default.
- Name files that define classes or components in PascalCase.
- Prefer named exports; use default exports only when a tool or framework
  requires them.
- Prefer TypeScript `type` aliases over `interface` declarations unless an
  interface is required.
- Do not use TypeScript `function` declarations; use arrow function
  expressions. Use `function` only when a tool, framework, overload, generator,
  `this` binding, or hoisting requirement makes it necessary.
- Define reusable enum-like TypeScript value sets as TypeScript string enums
  and validate them through Zod schemas built from `enumValues(...)`, instead
  of duplicating string literal unions or inline `z.enum([...])` arrays.
- Parse enum-like values that cross CLI, file, prompt, config, or other
  untrusted runtime boundaries with Zod before using them.
- Use single quotes for strings outside JSX. Use double quotes for JSX and HTML
  attributes unless that is not practical.
- Use trailing commas wherever the syntax allows them to reduce future diffs.
- Keep line width at 140 columns, not 80.
- Always use semicolons.
- Use Bun for local dependency installation, lockfile updates, and package
  verification commands when working inside this repository. Keep `bun.lock` as
  the repository lockfile.
- Run Biome through the package scripts when editing JavaScript, TypeScript,
  JSON, or JSONC files.
- After local package verification, use `bun run clean` to remove generated
  package artifacts. Do not use it as a dependency reset, and do not remove
  `node_modules/` or private `.local/` state unless a human explicitly asks.

## Write For The Actual Reader

Keep each artifact scoped to the reader that will actually use it:

- `README.md` is for humans adopting or understanding the repo
- `AGENTS.md`, `lotus-local/`, and `lotus-linear/` are for agents executing
  work
- keep install and adoption guidance in `README.md`, not in machine-facing
  workflow files
- keep machine-facing files operational; avoid self-descriptions and historical
  narration
- if a file is meant to be executed or followed by an agent, write only the
  rules and expectations needed at execution time

## What To Update Together

If you change naming, paths, or adoption flow, review at least:

- `README.md`
- `.codex-plugin/plugin.json`
- `lotus-local/`
- `lotus-linear/`
- any affected skill directories under those package roots

## Branch And Pull Request Hygiene

- Branch names for implementation work must use exactly one of these patterns:
  `feat/<taskid>`, `fix/<taskid>`, `chore/<taskid>`, or
  `epic/<epictaskid>`
- `<taskid>` and `<epictaskid>` must be the lowercase Linear issue identifier,
  for example `feat/max-77`, `chore/max-82`, or `epic/max-72`
- reject other branch formats, including Linear-generated or personal-prefix
  names such as `maxie/max-77-...`, descriptive slug branches, and
  `feature/<taskid>`
- pull request titles must use
  `<TASKID>: <short Linear-aligned description>`, for example
  `MAX-77: Implement repair and forced reinstall guidance`
- use `.github/PULL_REQUEST_TEMPLATE.md` when opening GitHub pull requests and
  fill every applicable section
- write PR titles, descriptions, implementation notes, verification notes, and
  risks in English
- open implementation PRs ready for review by default; use draft PRs only when
  the human explicitly asks for a draft

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MrMaxie/lotus-agents](https://github.com/MrMaxie/lotus-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
