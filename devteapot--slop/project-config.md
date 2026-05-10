---
trigger: always_on
description: - This repository is a multi-language monorepo for the SLOP protocol and SDKs.
---

# Agent Guide

- This repository is a multi-language monorepo for the SLOP protocol and SDKs.
- Primary stacks are TypeScript/Bun, Go, Rust, Python, plus app/example code.
- Prefer the smallest correct change and keep behavior aligned across SDKs when touching protocol logic.
- `spec/` and `website/docs` are important source-of-truth areas, not just marketing content.

## Repo-Specific Agent Rules

- No repo-local Cursor rules were found in `.cursor/rules/` or `.cursorrules`.
- No repo-local Copilot instructions were found in `.github/copilot-instructions.md`.
- Do not assume ESLint, Prettier, Biome, Ruff, or golangci-lint are enforced here unless you add them intentionally.
- Existing enforcement is mostly build, typecheck, generated-content checks, and tests.

## Workspace Basics

- Use `bun` for root workspace installs and most TypeScript tasks.
- Root workspace packages live under `packages/typescript/`, plus apps, websites, examples, and benchmarks.
- Python SDK lives in `packages/python/slop-ai`.
- Go SDK lives in `packages/go/slop-ai`.
- Rust SDK lives in `packages/rust/slop-ai`.
- Desktop app frontend is in `apps/desktop`; native Tauri code is in `apps/desktop/src-tauri`.
- Chrome extension is in `apps/extension`.
- CLI inspector is Go code in `apps/cli`.

## Core Commands

- Install workspace JS dependencies: `bun install --frozen-lockfile`
- Root TypeScript build: `bun run build`
- Root TypeScript tests: `bun run test`
- List scoped checks for current changes: `bun run preflight --list`
- Run scoped checks for current worktree: `bun run preflight`
- Run scoped checks since a base ref: `bun run preflight --since origin/main`
- Run scoped checks for explicit paths: `bun run preflight --files path/to/file another/path`
- There is no repo-wide `lint` script or repo-wide TS `typecheck` script.

## Single-Test Recipes

- Single Bun test file: `cd packages/typescript/sdk/core && bun test __tests__/tree-assembler.test.ts`
- Another Bun single file example: `cd packages/typescript/adapters/react && bun test __tests__/use-slop.test.tsx`
- TanStack Start single test file: `cd examples/full-stack/tanstack-start && bunx vitest run path/to/test-file.test.ts`
- One Python test file: `cd packages/python/slop-ai && python -m pytest tests/test_server.py`
- One Python test case: `cd packages/python/slop-ai && python -m pytest tests/test_server.py -k test_invoke`
- One Go test: `cd packages/go/slop-ai && go test ./... -run '^TestInvoke$'`
- One Rust test: `cd packages/rust/slop-ai && cargo test test_register_static`
- Root `bun run test` is a loop over publishable TS packages; it is not the right command for a single test.
- If you need a single test case, run the package-local runner and narrow further with Bun test-name filtering if needed.

## App And Site Commands

- Extension typecheck: `cd apps/extension && bun run typecheck`
- Extension build: `cd apps/extension && bun run build`
- Desktop frontend typecheck: `cd apps/desktop && bun run typecheck`
- Desktop web bundle build: `cd apps/desktop && bun run vite:build`
- Desktop full Tauri dev app: `cd apps/desktop && bun run dev`
- Desktop full Tauri build: `cd apps/desktop && bun run build`
- Docs generated-content check: `cd website/docs && bun run check:content`
- Docs build: `cd website/docs && bun run build`
- Landing/demo/playground build pattern: `cd website/<site> && bun run build`
- TanStack Start example build/test: `cd examples/full-stack/tanstack-start && bun run build` and `bun run test`
- Benchmarks: `cd benchmarks/mcp-vs-slop && bun run bench`

## Python Commands

- Install Python SDK in editable mode with test extras: `python -m pip install -e "packages/python/slop-ai[all]"`
- Run Python SDK tests from repo root: `python -m pytest packages/python/slop-ai/tests`
- Run Python SDK tests from package dir: `cd packages/python/slop-ai && python -m pytest tests`
- There is no checked-in Ruff or Black config here.

## Go Commands

- Build Go CLI inspector: `cd apps/cli && go build -o slop-inspect .`
- Run Go CLI inspector: `cd apps/cli && go run .`
- Run Go SDK tests: `cd packages/go/slop-ai && go test ./...`
- Use `go test` as both test and basic compile validation.
- There is no checked-in golangci-lint config.

## Rust Commands

- Run Rust SDK tests: `cd packages/rust/slop-ai && cargo test`
- Build Rust SDK: `cd packages/rust/slop-ai && cargo build`
- Build desktop native side directly if needed: `cd apps/desktop/src-tauri && cargo build`
- CI mainly validates Rust with `cargo test` in `packages/rust/slop-ai` and Tauri build via `apps/desktop`.

## Change-Validation Expectations

- Run the narrowest commands that cover the files you changed.
- If you touch a publishable TS package, at minimum run that package's `bun run build` and relevant `bun test` commands.
- Package-local TS command pattern: `cd packages/typescript/<group>/<pkg> && bun run build` and `bun test`
- If you touch shared protocol behavior, inspect sibling SDKs and update tests/docs as needed.
- If you touch `spec/` or docs generation inputs, run `cd website/docs && bun run check:content`.
- If you touch desktop frontend code, run `cd apps/desktop && bun run typecheck`.
- If you touch extension code, run `cd apps/extension && bun run typecheck`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devteapot/slop](https://github.com/devteapot/slop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
