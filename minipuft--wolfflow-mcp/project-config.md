---
trigger: always_on
description: **Source of Truth**: `server/dist/**`. Confirm behavior there before describing or modifying functionality.
---

# Claude Prompts MCP -- Operator Handbook

**Source of Truth**: `server/dist/**`. Confirm behavior there before describing or modifying functionality.

## Core Principles

1. **MCP Tooling Only** -- Prompts, templates, chains flow through MCP tools. Manual edits under `server/prompts/**` forbidden.
2. **Contracts as SSOT** -- Schemas generated from `tooling/contracts/*.json`. Run `npm run generate:contracts`, never edit `_generated/`.
3. **Transport Parity** -- Runtime changes must work in STDIO and SSE.
4. **Docs/Code Lockstep** -- Update relevant doc in `docs/` when behavior changes.
5. **Validation Discipline** -- `npm run typecheck && npm run lint:ratchet && npm run test:ci` minimum. Add `validate:arch` for module boundaries.

## Node.js Support Boundaries

| Surface | Supported Node.js | Enforcement |
|---------|-------------------|-------------|
| MCP server and desktop extension | >=22.13.0 | `server/package.json`, `manifest.json`, CI on 22.13.0 and 24 |
| Standalone CPM CLI | >=18.18.0 | `cli/package.json` and CLI runtime validation |
| Local development and publishing | 24 | `.node-version` and publish workflows |

The server floor is where `node:sqlite` is available without an experimental flag. The standalone CLI remains a separate, self-contained compatibility surface.

## Validation Gates (one contract, impact-aware subsets)

**CI is the contract; every other gate is a documented strict subset of it.**

The three gates once ran three different suites with no subset relation, so a green
`pre-push` did not predict CI and `validate:full` did not either -- that is how a pyrefly
failure reached `main` from a clean local push.

`scripts/classify-validation-scope.js` is the changed-path SSOT for local push and CI.
It recognizes two narrow safe scopes and sends every empty, mixed, executable,
configuration, dependency, deleted-unknown, or unrecognized change to `full`.

| Scope | Trigger | Pre-push | CI |
|------|---------|----------|----|
| `docs` | documented root handbooks, `docs/**/*.md`, `plans/**/*.md`, and server/CLI READMEs only | changed-line hygiene + Prettier on existing changed files | classifier hygiene; four protected jobs report intentional lightweight passes |
| `hooks` | only `hooks/**` plus optional docs | docs checks + `validate:python` | pinned Ruff/Pyrefly/Pytest/PyYAML; other protected jobs report intentional lightweight passes |
| `full` | everything else; empty/unknown input | typecheck · lint ratchet · format · conditional Python · unit tests · architecture · versions · build | typecheck · `validate:all` · CLI · build/smoke/schema · Node 22/24 unit/coverage/integration/E2E |

The CI workflow remains unconditional. Do not add workflow-level `paths` or
`paths-ignore`: a required workflow skipped before jobs exist leaves its context
pending. Routing happens inside the workflow while the literal `Lint & Validate`,
`CLI`, `Build`, and `Test Suite` job names remain stable.

`.husky/pre-commit` remains the fast contract-regeneration, staged-lint, conditional
Python, lint-ratchet, and typecheck gate. Every local route remains a subset of CI.

**Adding a step to a hook that CI does not run breaks the contract** -- add it to
`validate:all` first, which CI runs whole. Removing a step CI depends on breaks it too.
\* conditional on `hooks/` changes.

Formatting is covered by `validate:format` in the full route. `pre-push` checks existing
repo-level JSON/MD/YAML files in the push range. Anything a generator owns belongs in
`.prettierignore` with a reason -- otherwise the generator and Prettier disagree.

## Documentation Map

| Topic | Doc |
|-------|-----|
| Architecture & runtime | `docs/architecture/overview.md` |
| MCP tools & symbolic commands | `docs/reference/mcp-tools.md` |
| Prompt authoring | `docs/tutorials/build-first-prompt.md` |
| Chains lifecycle | `docs/concepts/chains-lifecycle.md` |
| Gates | `docs/guides/gates.md` |
| Injection control | `docs/guides/injection-control.md` |
| Identity & scope | `docs/guides/identity-scope.md` |
| Skills Sync | `docs/guides/skills-sync.md` |
| Telemetry & observability | `docs/guides/telemetry-observability.md` |
| Troubleshooting | `docs/guides/troubleshooting.md` |
| Contributing & PR process | `CONTRIBUTING.md` |
| README charter (root README authoring rules) | `docs/portfolio/readme-charter.md` |
| Release highlights | `CHANGELOG.md` |

Read the relevant doc before editing. Update docs when behavior changes.

## Command Reference (run inside `server/`)

| Command | Purpose |
|---------|---------|
| `npm run build` | esbuild bundle -> `dist/index.js` |
| `npm run verify:mcp` | Spawn a server from `dist/` and prove all 3 MCP tools answer — **use instead of restarting Claude Code to check a build**. Refuses to run against a stale `dist/` |
| `npm run typecheck` | Strict TS type validation — **`src/` only**, `tsconfig.json` excludes `tests/` |
| `npm test` | Full Jest suite |
| `npm run lint:ratchet` | Fail if ESLint violations increased |
| `npm run typecheck:tests:ratchet` | Fail if `tests/` type errors increased. Covers the call sites `typecheck` cannot see — a constructor change can otherwise land green against a test file that no longer compiles |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minipuft/wolfflow-mcp](https://github.com/minipuft/wolfflow-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
