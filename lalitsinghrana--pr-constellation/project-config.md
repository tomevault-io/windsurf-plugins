---
trigger: always_on
description: > Durable instructions for agents maintaining this repository. Apply the
---

# PR Constellation Engineering Guidelines

**Version 1.1.0**

> Durable instructions for agents maintaining this repository. Apply the
> repository-wide JavaScript rules to every JavaScript file, then apply the
> React or Node.js rules when that runtime is involved.

---

## Abstract

This guide turns the project's engineering sources into decisions an agent can
apply while editing code. It favors the smallest cohesive design that preserves
behavior, exposes clear ownership, validates external input, and remains easy to
test. Generic advice never justifies rewriting working project conventions.

Rules are ordered by scope:

1. **General JavaScript** applies to all `.js`, `.mjs`, and `.jsx` files.
2. **React** additionally applies to components, hooks, pages, and browser UI.
3. **Node.js** additionally applies to the server, CLI, analysis workflows,
   build/render code, and Node-run tests.

When rules compete, protect correctness, security, data, and accessibility
first; then preserve existing product behavior and public contracts; then
prefer the repository's established pattern over a generic source. Do not
re-architect code merely to make it resemble an example application.

---

## Table of Contents

1. [Project Workflow](#project-workflow)
2. [General JavaScript](#1-general-javascript)
3. [React](#2-react)
4. [Node.js](#3-nodejs)

---

## Project Workflow

### Read before changing

**Impact: CRITICAL**

- Trace the affected flow end to end and inspect every caller before changing a
  shared function, hook, component, or module contract.
- Read the owning area's `AGENTS.md` (`client/AGENTS.md`, `server/AGENTS.md`,
  `analysis-worker/AGENTS.md`) before changing that area.
- Search for an existing helper, component, dependency, test pattern, and
  neighboring convention before introducing a new one.
- Keep changes local to the owning feature. Do not add a top-level directory,
  architectural layer, dependency, or reusable abstraction for a hypothetical
  future use.

### Dependencies and UI primitives

**Impact: HIGH**

- Prefer, in order: the language or platform, an existing project utility, an
  installed dependency, then a maintained package when it replaces meaningful
  generic behavior such as URL state, date parsing, or form validation.
- Before creating a UI primitive, inspect `client/src/components/ui`, the shadcn
  registry, and maintained packages. Hand-write a generic primitive only when
  none meets the interaction and accessibility requirements.
- Add shadcn components from the repository root with
  `pnpm exec shadcn add <component>` so `components.json` remains the single UI
  registry.

### Verification

**Impact: CRITICAL**

- Run the narrowest relevant check while iterating. `pnpm test` runs every
  `node:test` suite serially; a specific `*.test.js` or `*.test.mjs` file is the
  usual focused check.
- Run `pnpm check` before handing off a cross-cutting or completed change. It
  checks Biome formatting and lint diagnostics, runs all tests, and builds the
  client.
- Prefer `pnpm exec biome format --write <touched-paths>` during focused work,
  and do not include unrelated formatting churn.
- For UI work, verify the relevant page at its real localhost route. Inspect
  errors and console output, and capture a screenshot when layout or visual
  state changed.
- Test the behavior most likely to regress, including a failure path when the
  change affects validation, persistence, processes, or network I/O.

### Generated reviews and local URLs

**Impact: HIGH**

- Generated review runs belong under the gitignored `.reviews/` directory. Do
  not commit them unless explicitly requested.
- Serve user-facing reviews with `pnpm dev` on the fixed port. Reuse a server
  already serving this workspace on port `4397`, or stop it before starting a
  new one; do not switch to a random port.
- Hand off the stable route
  `http://127.0.0.1:4397/reviews/<review-slug>/`, not a `file://` or timestamped
  URL. Historical revisions remain under
  `/reviews/<review-slug>/<run-id>/`.

---

## 1. General JavaScript

### 1.1 Keep code with its owner

**Impact: HIGH**

Use the existing structure rather than inventing parallel homes:

- `client/` owns the PR Constellation UI, generated React Flow review pages, renderer,
  Shiki integration, shared shadcn primitives, and browser-facing tests.
- `server/` owns the long-running HTTP server, GitHub synchronization,
  dashboard API, queue coordination, persistence, and server tests.
- `analysis-worker/` owns the CLI, run coordinator, PR fetching, diff
  inventory, prompts, schemas, validation, judging, retries, and analysis tests.
- `analysis-worker/workflow/` preserves the numbered headless stages. Keep
  stage-specific code, prompts, schemas, tests, and documentation with their
  current stage.
- Root files such as `package.json`, `biome.json`, `components.json`, and
  `jsconfig.json` configure all three areas.

These are ownership boundaries in one Node and pnpm project, not independent
packages or workspaces. Keep tests in the owning subproject's `tests/`
directory. `client/src/review/render.js` remains client-owned because it builds
the client artifact even though Node executes it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LalitSinghRana/pr-constellation](https://github.com/LalitSinghRana/pr-constellation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
