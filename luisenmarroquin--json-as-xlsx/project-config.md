---
trigger: always_on
description: Guidance for AI coding assistants (e.g. Claude Code) when working in this repository.
---

# Repository guidance for AI assistants

Guidance for AI coding assistants (e.g. Claude Code) when working in this repository.

## Mandatory Synchronization

- `AGENTS.md` and `CLAUDE.md` must say exactly the same thing.
- If an assistant modifies one of these files, it must apply the same change to the other in the same task.
- After editing them, verify from the repository root with `cmp -s AGENTS.md CLAUDE.md`.
- Do not leave instructions, notes, preferences, or knowledge updates only in one of the two files.

## Concurrent AI Work — IMPORTANT

- Multiple AI assistants may work in this repository at the same time.
- If you notice changes you did not make, do **not** edit, revert, stage, or
  otherwise disturb them unless Luis explicitly asks you to handle those exact
  changes. They likely belong to another AI assistant working in parallel.
- If those parallel changes affect your task, inspect them enough to avoid
  conflicts and work around them. Ask Luis before proceeding only if they make
  your task impossible or unsafe.

## Project

`json-as-xlsx` — a Yarn + Lerna monorepo. Packages live in `packages/*`:

- `main-library` — the published npm package `json-as-xlsx` (this is what gets released).
- `demo-express` — runnable Express example (`ts-node`, started with `yarn start`).
- `demo-reactjs` — the web UI / demo site (built with Vite).

Validation commands (run from the repo root):

- `yarn test` — runs the library Jest suite.
- `yarn build` — builds `main-library` (tsc + uglify) and `demo-reactjs`.
- `yarn static` — copies the built `demo-reactjs` site into `build/` for deploy.

Use the Node version in `.nvmrc` (currently `24.18.0`). `.node-version` pins the
same version — always update both together: GitHub Actions reads `.nvmrc`
(`node-version-file`) and Cloudflare Pages' build system reads `.node-version`
(in practice it ignores `.nvmrc`). Keep it on a version supported by the
toolchain (lerna 9 needs `^20.19 || ^22.12 || >=24`).

TypeScript is intentionally pinned to `~6.0.3` (6.0.x only, matching the tilde
pin used across the maintainer's other repos) in every package — do not bump it
to 7.x yet. ts-jest still needs the legacy JS compiler API (e.g.
`ts.sys.fileExists`) that TypeScript 7 (the Go-native compiler, released
2026-07-08) removed. Retry the upgrade once ts-jest supports TS 7; everything
else stays on latest.

## Repository language — IMPORTANT

- English is the language of this repository.
- All repository artifacts must be written in English, including code,
  comments, documentation, commit messages, pull request titles/descriptions,
  issue comments, pull request review replies, release notes, examples, and
  user-facing copy in the demos.
- Do not add Spanish text to repository files or GitHub comments unless Luis
  explicitly asks for Spanish text for a specific user-facing purpose.

## Demo parity — IMPORTANT

`demo-reactjs` (the web UI) and `demo-express` (the API) are two views of the
same library and **must demonstrate the same features**. Whenever you add or
change an example in one, make the equivalent change in the other in the same
task:

- If you add an example/download to the UI
  (`packages/demo-reactjs/src/App.tsx`), add the matching endpoint to the API
  (`packages/demo-express/src/server.ts`) — and the other way around.
- Keep the demonstrated data and styling equivalent so the two stay in sync.

## Demo dependency workflow — IMPORTANT

- The demo packages are Yarn workspaces and are expected to be installed and run
  from the repository root, not as standalone packages from inside
  `packages/demo-*`.
- Keep the demos' `json-as-xlsx` dependency pinned to the `main-library` package
  version being prepared for release. From the repo root, Yarn resolves that
  dependency to the local `packages/main-library` workspace.
- Do **not** switch the demo dependencies back to `file:../main-library` just to
  support standalone installs before the version is published to npm. That can
  create stale local path installs or nested package copies, and makes the demos
  less clearly reflect the release candidate. Only change this if Luis
  explicitly asks to support standalone demo installs.

## Backward compatibility — IMPORTANT

`json-as-xlsx` is a published library that other people depend on. **The
`main-library` public API MUST stay backward-compatible** so that existing
users' code keeps working when they upgrade:

- **Never remove or rename** an exported function, type, option, or parameter,
  and never change the meaning, default value, or return type of an existing
  one.
- **Add, don't change.** New behavior must be opt-in (e.g. a new optional
  setting) with defaults that preserve the current behavior.
- This covers everything exported from `packages/main-library/src/index.ts`
  (e.g. `xlsx`, `IJsonSheet`, `ISettings`, `IColumn`, the `utils` re-export) and
  the accepted shape of the `data` / `settings` objects.
- **If a change would be breaking, STOP — do NOT make it. Tell Luis first**,
  explain exactly what would break and why, and wait for his explicit approval.
  Breaking changes are only allowed with his go-ahead (and a major version bump).

## Branch workflow — IMPORTANT


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuisEnMarroquin/json-as-xlsx](https://github.com/LuisEnMarroquin/json-as-xlsx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
