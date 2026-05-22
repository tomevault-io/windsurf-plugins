---
trigger: always_on
description: This file is for coding agents working in this repository.
---

# AGENTS.md

This file is for coding agents working in this repository.

## Repository Shape

- Monorepo managed with Bun workspaces and Turbo.
- Main packages: `apps/trenchclaw`, `apps/runner`, `apps/frontends/gui`, `website`, `apps/types`.
- Tests live mostly in the top-level `tests/` directory.
- Bun test preloads `tests/setup.ts` via `bunfig.toml`.

## Product Mental Model

- TrenchClaw is a local runtime first.
- `apps/trenchclaw` owns settings, state, tools, policy, storage, and execution.
- `apps/frontends/gui` is a client of that runtime, not the storage authority.
- `apps/runner` packages and launches the runtime plus GUI.
- Runtime state is instance-scoped. Think in terms of one active instance, not one shared global workspace.
- `.runtime/` is the tracked contract and seed area. Mutable state belongs in the runtime-state root outside normal repo editing flows.

## Source Of Truth Docs

- `ARCHITECTURE.md` is the canonical architecture document.
- `website/src/content/shared/architecture.md` is generated from `ARCHITECTURE.md` by `bun run --cwd website content:sync`.
- Authored website docs live in `website/src/content/docs/`.
- Install bootstrap wrappers live in `website/static/install/`.
- The canonical runtime installer logic lives in `scripts/install-trenchclaw.sh`.

## Rule Files

- No repo-level `.cursorrules` file was found.
- No `.cursor/rules/` directory was found.
- No `.github/copilot-instructions.md` file was found.
- Follow this file plus the existing code patterns in the repo.

## Core Commands

- Install deps: `bun install`
- Run all lint/typecheck/test/build: `bun run ci`
- Run full monorepo lint/typecheck/test: `bun run lint`, `bun run typecheck`, `bun run test`
- Run full monorepo build: `bun run build:all`
- Run stronger verification pass: `bun run check:all`
- Run strongest verification including chat smoke: `bun run check:all:with-chat`

## App / Runtime Commands

- Build packaged app bundle: `bun run app:build`
- Start runner directly: `bun run app:start`
- Build and launch packaged app: `bun run launch`
- Launch release-style app: `bun run launch:release`
- Start dev bootstrap flow: `bun run dev`
- Initialize external dev runtime: `bun run dev:runtime:init`
- Clone instance state into dev runtime: `bun run dev:instance:clone -- --from-root <src> --to-root ~/.trenchclaw-dev-runtime --from-instance 00 --to-instance 00 --parts wallets,db,settings`
- Clean build artifacts: `bun run cleanup:build`

## Core Package Commands

Run from repo root unless noted.

- Core lint: `bun run --cwd apps/trenchclaw lint`
- Core lint fix: `bun run --cwd apps/trenchclaw lint:fix`
- Core typecheck: `bun run --cwd apps/trenchclaw typecheck`
- Core tests: `bun run --cwd apps/trenchclaw test`
- Core build: `bun run --cwd apps/trenchclaw build`
- Refresh generated context/knowledge: `bun run --cwd apps/trenchclaw generate`
- Start runtime server only: `bun run --cwd apps/trenchclaw runtime:start`

## Runner Commands

- Runner lint: `bun run --cwd apps/runner lint`
- Runner lint fix: `bun run --cwd apps/runner lint:fix`
- Runner typecheck: `bun run --cwd apps/runner typecheck`
- Runner build: `bun run --cwd apps/runner build`

## GUI Commands

- GUI dev via root bootstrap: `bun run gui:dev`
- GUI standalone dev server: `bun run --cwd apps/frontends/gui dev:standalone`
- GUI build/lint/typecheck: `bun run frontend:build`, `bun run frontend:lint`, `bun run frontend:typecheck`
- GUI package-local lint: `bun run --cwd apps/frontends/gui lint`
- GUI package-local typecheck: `bun run --cwd apps/frontends/gui typecheck`

## Website Commands

- Website dev: `bun run website:dev`
- Website lint/test/typecheck: `bun run website:lint`, `bun run website:test`, `bun run website:typecheck`
- Website Svelte check: `bun run website:svelte-check`
- Website build: `bun run website:build`
- Website full CI pass: `bun run website:ci`
- Website content sync only: `bun run --cwd website content:sync`

## Docs Workflow

- Update `ARCHITECTURE.md` when the product architecture changes.
- Run `bun run --cwd website content:sync` after editing `ARCHITECTURE.md`.
- Edit website-authored docs only in `website/src/content/docs/`.
- Do not hand-edit `website/src/content/shared/architecture.md`; it is generated.
- Keep docs concise, concrete, and product-facing. Prefer clear statements over speculative caveats or internal debate.

## Test Commands

- All tests / app-focused CI / website-only: `bun run test`, `bun run appci`, `bun run website:test`
- Runtime chat focused suite: `bun run test:runtime-chat`
- Launch chat smoke: `bun run test:launch-chat`

## Running One Test File

- Single top-level test file: `bun test tests/runtime/chatService.test.ts`
- Another example: `bun test tests/website/smoke.test.ts`
- Multiple explicit files: `bun test tests/runtime/chatService.test.ts tests/frontends/transport.test.ts`
- From `website/`, website tests use relative paths like: `bun test ../tests/website`

## Running One Test or Describe Block

- Bun supports test-name filtering.
- Single named test example: `bun test tests/runtime/chatService.test.ts -t "maps timeout failures to explicit runtime chat errors"`
- Prefer file + name filtering together for speed and determinism.

## Lint / Typecheck Expectations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henryoman/TrenchClaw](https://github.com/henryoman/TrenchClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
