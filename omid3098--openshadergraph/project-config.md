---
trigger: always_on
description: <!-- OPENSPEC:START -->
---

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# OpenShaderGraph – Working Agreement

Purpose: Keep agents aligned on the absolute essentials needed to build, test, and ship the TypeScript-first OpenShaderGraph. Be brief. Be consistent. Ship green. Focus on clean, maintainable and extensible code.

## Tech Stack

- Runtime: `bun` (package manager + task runner)
- Core + UI: TypeScript, ShadCN for UI components and ReactFlow for graph rendering
- Tests: `vitest` (unit), `playwright` (E2E/visual)
- Lint/Format: ESLint (treat warnings as failures)
- Docs via MCP: Context7 (for up-to-date library APIs)

## Canonical Data (Single Source of Truth)

- Nodes: `data/nodes/**.json`
- Base node schema: `data/node.json`
- Language packs: `data/languages/*.json`

Minimal graph rules:

- IDs are integers unique within the graph hierarchy.
- Connections encode as relative refs: `../<nodeId>/<pinId>` on both ends (input and output).
- Never reorder pins, children, or IDs during load/save round-trips.

## Required Gates (run ALL locally before confirming a solution, adding a feature, or pushing)

- Agents MUST execute `bun run gates` **before committing**. This umbrella task sequentially runs lint, typecheck, unit tests, coverage, shader validation, and Playwright E2E checks, ensuring every gate is exercised in one command.
- If `bun run gates` surfaces a failing subcommand, fix the issue and rerun the full command until it passes. Do **not** rely on running individual gate commands in isolation.
- First time only: run `bun run test:e2e:install` before `bun run gates` so Playwright has the required browsers.
- `bun install --frozen-lockfile` remains required whenever dependencies change.

Agent behavior requirements:

- Attach the full stdout/stderr for each command to the PR (as artifacts or a comment). Logs should include the exact Bun version and the commands run.
- If any command exits non-zero, the agent MUST abort the change, mark the PR as failing, and include a remediation plan.
- Agents MUST call `todo_write` (merge=true) to record each gate's status (pending → in_progress → completed/failed) before approving or merging. The todo entries should be one-line, verb-led items matching the repo `todo_spec` style.
- Agents MUST not merge or approve PRs on behalf of humans. They may propose changes and create PRs, but approval/merge requires a human reviewer after gates pass in CI.
- Agents SHOULD pin/declare the Bun runtime version used for validation and prefer reproducing the CI Bun version (from `bun.lock` or workflow) to avoid environment drift.

Enforcement & recommended infra:

- CI MUST include a fast-fail job that runs `bun run lint` and `bun x tsc` and be marked as a required status check in branch protection rules.
- PR templates should include an "Agent Validation" checklist and require attached logs/artifacts for lint/tsc/tests when an agent created the PR.
- Consider adding `husky` + `lint-staged` to enforce local pre-push checks for contributors (human or agent-run workflows).

These requirements are mandatory for any automation labeled as an "agent" in PR metadata. Failure to follow these rules will cause automated validations to mark the PR as non-compliant and block merges.

### E2E Testing Setup

First time only - install Chromium browser:

```bash
bun run test:e2e:install
```

Run E2E tests (Chromium - matches CI):

```bash
bun run test:e2e
```

For local development with VSCode Playwright extension:

- Use Testing sidebar to run/debug individual tests
- Tests run against production server (`bun run start`)
- See `e2e/README.md` for detailed guide

## MCP Docs (Context7) Usage

- Resolve library first, then fetch focused docs.
- Typical targets: `reactflow`, `bun`, `@playwright/test`, `vitest`.
- Keep token caps reasonable and request specific topics (e.g., "parent/child nodes", "edges API").

## Development Rules

- ALWAYS Tests first: add/modify unit tests in `src/core/**` and E2E tests in `e2e/**` for all new features and bug fixes.
- Data integrity next: adhere to `data/node.json` and language packs; fail safe with clear errors on unknown/missing templates.
- Small, surgical diffs; prefer targeted fixes over broad refactors.
- When asked to create new nodes, make sure to put it in the right directory and also create correct templates in data/languages for all avaibale languages.
- When adding core features, extend the TypeScript core under `src/core/**` and keep UI as a thin consumer.
- Preview source of truth: preview panel always renders using a ThreeJS GLSL fragment shader compiled from the current graph. Always compile `ThreeJS_GLSL` under the hood for preview, regardless of the selected output language.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omid3098/openshadergraph](https://github.com/omid3098/openshadergraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
