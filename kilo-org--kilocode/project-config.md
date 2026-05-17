---
trigger: always_on
description: Kilo CLI is an open source AI coding agent that generates code from natural language, automates tasks, and supports 500+ AI models.
---

# AGENTS.md

Kilo CLI is an open source AI coding agent that generates code from natural language, automates tasks, and supports 500+ AI models.

- ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.
- The default branch in this repo is `main`.
- Prefer automation: execute requested actions without confirmation unless blocked by missing info or safety/irreversibility.
- You may be running in a git worktree. All changes must be made in your current working directory — never modify files in the main repo checkout.

## Build and Dev

- **Dev**: `bun run dev` (runs from root) or `bun run --cwd packages/opencode --conditions=browser src/index.ts`
- **Dev with params**: `bun dev -- help`
- **Extension**: `bun run extension` (build + launch VS Code with the extension in dev mode). Pass `--no-build` to skip the build.
- **Typecheck**: `bun turbo typecheck` (uses `tsgo`, not `tsc`)
- **Test**: `bun test` from `packages/opencode/` (NOT from root -- root blocks tests)
- **Single test**: `bun test ./test/tool/tool-define.test.ts` from `packages/opencode/`
- **CLI build artifact size check**: after `bun run script/build.ts --single --skip-install` in `packages/opencode/`, use `du -h dist/*/*/bin/kilo` (scoped package output lives under `dist/@kilocode/`)
- **SDK regen**: After changing server endpoints in `packages/opencode/src/server/`, run `./script/generate.ts` from root to regenerate `packages/sdk/js/`
- **Knip** (unused exports): `bun run knip` from `packages/kilo-vscode/`. CI runs this — all exported types/functions must be imported somewhere. Remove or unexport unused exports before pushing.
- **Source links**: After adding or changing URLs in `packages/kilo-vscode/`, `packages/kilo-vscode/webview-ui/`, or `packages/opencode/src/`, run `bun run script/extract-source-links.ts` from the repo root and commit the updated `packages/kilo-docs/source-links.md`. CI runs this check — the build fails if the file is stale.
- **kilocode_change check**: `bun run check-kilocode-change` from `packages/kilo-vscode/`. CI runs this — `kilocode_change` is a marker for upstream merge conflicts and must not appear in `packages/kilo-vscode/` or `packages/kilo-ui/` (these are entirely Kilo Code additions). Remove the markers before pushing.
- **opencode annotation check**: `bun run script/check-opencode-annotations.ts` from repo root. CI runs this on PRs touching `packages/opencode/` — every Kilo-specific change in shared opencode files must be annotated with `kilocode_change` markers. Exempt paths (no markers needed): `packages/opencode/src/kilocode/`, `packages/opencode/test/kilocode/`, and any path containing `kilocode` in the name.
- **workflow allowlist**: `bun run script/check-workflows.ts` from repo root. CI runs this as part of the annotations workflow — any `.yml` / `.yaml` file added to or removed from `.github/workflows/` must be reflected in the hardcoded list in `script/check-workflows.ts`. Prevents upstream-merged workflows from silently starting to run in our CI.
- **Backend/SDK programmatic testing**: see [TESTING.md](./TESTING.md) for spawning the local main-branch backend (`bun dev serve`) and driving it via `curl` — use this instead of `kilo serve` (prod binary) when testing backend fixes.

## Quality Checks

Before saying an implementation is ready, run the smallest relevant checks that can catch lint, typecheck, and test failures for the touched package. Do not rely on manual extension launch to discover build problems. Fix failures you introduced before the final response, or state exactly which check is still failing or could not be run.

| Area | Checks |
|---|---|
| Root / cross-package | `bun run lint`, `bun run typecheck` |
| CLI | From `packages/opencode/`: `bun run typecheck`, `bun test` or targeted `bun test ./path/to/file.test.ts` |
| VS Code extension | From `packages/kilo-vscode/`: `bun run typecheck`, `bun run lint`, `bun run test:unit` or `bun run test` |
| Extension build/package | From `packages/kilo-vscode/`: `bun run compile` or `bun run package` when touching build, packaging, SDK, or webview integration paths |
| CI-only guards | Run affected guards documented above, such as `bun run knip`, `bun run check-kilocode-change`, `bun run script/check-opencode-annotations.ts`, or source link extraction |

Never run root `bun test`; the root script prints `do not run tests from root` and exits with code 1. Use package-level tests instead.

## Products

All products are clients of the **CLI** (`packages/opencode/`), which contains the AI agent runtime, HTTP server, and session management. Each client spawns or connects to a `kilo serve` process and communicates via HTTP + SSE using `@kilocode/sdk`.

| Product | Package | Description |
|---|---|---|
| Kilo CLI | `packages/opencode/` | Core engine. TUI, `kilo run`, `kilo serve`. Fork of upstream OpenCode. |
| Kilo VS Code Extension | `packages/kilo-vscode/` | VS Code extension. Bundles the CLI binary, spawns `kilo serve` as a child process. Includes the **Agent Manager** — a multi-session orchestration panel with git worktree isolation. |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
