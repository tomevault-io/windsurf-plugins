---
trigger: always_on
description: Operating instructions for coding agents working in this repository.
---

# AGENTS.md

Operating instructions for coding agents working in this repository.

## Project identity

- Project: `agenticflowx`
- Purpose: VSCode extension monorepo for AgenticFlowX (AFX)
- Package manager: `pnpm@10.32.1`
- Build orchestration: Turbo
- Language: TypeScript throughout
- Runtime targets:
  - `apps/vscode`: VSCode extension host, Node.js target, bundled with esbuild
  - `apps/chat`: VSCode side panel webview, browser target, React + Vite
  - `apps/workbench`: VSCode bottom panel webview, browser target, React + Vite

## Current stack

Root:

- pnpm workspaces via `pnpm-workspace.yaml`
- Turbo via `turbo.json`
- Shared TypeScript base config in `tsconfig.base.json`

Apps:

- `apps/vscode`
  - TypeScript
  - esbuild
  - `@types/vscode`
  - output: `out/`
- `apps/chat`
  - React 18
  - Vite 5
  - Tailwind 4 via `@tailwindcss/vite`
  - output: `dist/`
- `apps/workbench`
  - React 18
  - Vite 5
  - Tailwind 4 via `@tailwindcss/vite`
  - output: `dist/`

Packages:

- `packages/ui` → `@afx/ui`
  - Shared UI primitives, composites, design system
  - Meridian tokens at `src/tokens/meridian.css`
  - No standalone build step; consumed directly through TypeScript/Vite path aliases
- `packages/shared` → `@afx/shared`
  - Shared types, constants, message protocol
  - Structured `Logger` contract: leveled (silent..trace), scoped child loggers, lazy callbacks, pluggable sinks (see ADR-0003)
  - Pure TypeScript
- `packages/parsers` → `@afx/parsers`
  - Markdown/frontmatter/spec/task/journal parsers
  - Pure TypeScript
  - Uses `gray-matter`
- `packages/transport` → `@afx/transport`
  - Transport abstraction (VSCode postMessage, mock with 13 scenarios)
  - Enables browser-based dev loop without VSCode
  - Pure TypeScript + React-free
- `packages/agent/pi` → `@afx/agent-pi`
  - Pi coding-agent adapter — RPC transport (subprocess JSONL)
  - Implements `AgentManager` from `@afx/shared`
  - Node.js only — no VSCode imports; config injected via `PiRpcManagerOptions`
  - Naming is transport-explicit (`rpc-client.ts`, `rpc-manager.ts`)

## Commands

Run commands from the repository root unless noted.

```bash
pnpm install
pnpm check:types
pnpm build
pnpm dev
pnpm check:lint
pnpm clean
```

Useful package-specific commands:

```bash
pnpm --filter "./apps/vscode" build
pnpm --filter "apps/chat" build
pnpm --filter "apps/workbench" build
pnpm --filter "apps/chat" dev
pnpm --filter "apps/workbench" dev
```

Turbo filters use package names such as `apps/chat` and `apps/workbench`; use the path filter
`./apps/vscode` for the extension host because its package name is `agenticflowx`.

## Verification

> **Canonical commands** (see `docs/specs/430-dx-enforcement/430-dx-enforcement.md` [FR-1] [FR-2] [FR-34] [DES-API]).
> Replaces the legacy `ci` / `health` / `health:full` script trio.

### Two-tier verification surface

```text
pnpm verify         Fast lifecycle (~30–90s on M-series, warm cache).
                    Runs in parallel via turbo --continue (full failure list, not fail-fast):
                      check:types · check:lint · check:format · check:md · check:knip · test
                    Pre-push hook also runs this.
                    Use after every change.

pnpm verify:full    Full PR lifecycle. Runs `verify` then build + size-limit + e2e (chat + vscode).
                    What CI runs.
                    Use before merging.
```

### The verify → fix → verify loop

```text
pnpm verify         If it fails on auto-fixable issues …
pnpm fix            … runs prettier + markdownlint --fix + eslint --fix in that order.
pnpm verify         Re-run. Anything still failing is a real issue you must address.
```

`pnpm fix` does NOT auto-resolve:

- TypeScript type errors (`check:types` failures need code changes).
- ESLint architecture-boundary violations (`no-restricted-imports`).
- Missing `@see` JSDoc anchors required by spec-driven files.
- Coverage threshold breaches (need new tests, not formatting).
- Conventional Commit / commitlint failures (handled by `commit-msg` hook, not lint).

These remain `pnpm verify` failures and require human or LLM judgement.

### Targeted checks (when the full suite is overkill)

For one-off, in-flight work it's still fine to run the underlying scripts directly:

- Type-only changes: `pnpm check:types`
- Single app/package: `pnpm --filter "apps/chat" build` etc.
- Markdown only: `pnpm check:md` (or `pnpm check:md:fix` to auto-fix)

But **never report a task complete without running `pnpm verify` and reading the output.**

## Commit log conventions

Commit headers must follow `type(scope): imperative summary`, with a mandatory scope from `scripts/generate-scope-enum.mjs`. For non-trivial commits, use the AFX body shape from `.gitmessage` and `docs/specs/400-dx-conventions/`:

```text
Why:
- What problem this solves.

Changed:
- What changed, grouped by surface.

Spec:
- docs/specs/XXX-name/spec.md [FR-X]
- docs/specs/XXX-name/design.md [DES-X]

Traceability:
- @see retargeting, map IDs, generated artifacts, or none.

Verification:
- pnpm verify
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgenticFlowX/agenticflowx](https://github.com/AgenticFlowX/agenticflowx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
