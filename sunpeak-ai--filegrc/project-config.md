---
trigger: always_on
description: This monorepo builds filegrc, a Git-native GRC system for SOC 2 work. It has two Node.js packages:
---

# filegrc Repository Instructions

## Purpose

This monorepo builds filegrc, a Git-native GRC system for SOC 2 work. It has two Node.js packages:

- `filegrc`: the zero-dependency filegrc engine, which validates, searches, edits, and renders GRC data.
- `create-filegrc`: the filegrc scaffolder, which creates a standalone SOC 2 repository.

The generated repository is the product. Keep it understandable to an engineer who opens it without prior context.

## Agent-facing product surface

Treat headless use as a first-class interface. An agent with no filegrc context must be able to discover the right record type, inspect current relationship candidates, create or update JSON and Markdown through one validated payload, complete scheduled and event work, prepare an audit, and verify the result without opening the renderer.

- Keep the generated root `AGENTS.md` as the program and Git guide.
- Keep `data/AGENTS.md` as the universal record workflow. Add collection-level `AGENTS.md` files only where a wrong action has material compliance, privacy, or audit consequences.
- Keep `filegrc guide`, `types`, `list`, `get`, `references`, `scaffold`, CRUD, `content`, obligations, events, program readiness, audit readiness, and evidence packets model-driven.
- Scaffold files are prompts, not compliance facts. They must keep incomplete work in a non-final state and make missing required values obvious.
- Browser and CLI mutations must use the same domain functions and the same `{ record, content }` shape.
- Every resource type must pass automated guide and scaffold coverage. Test first-class multi-record workflows through the CLI as well as their domain functions.

## Product principles

- Git is the system of record. GRC records live as plain, reviewable files under `data/`.
- Git supplies file history, authors, timestamps, diffs, commit messages, and revision IDs.
- Domain events still need explicit dates. Do not replace dates such as `occurredOn`, `approvedOn`, or `completedOn` with Git metadata.
- Do not store a second change log or duplicate Git-derived fields such as `createdAt`, `updatedAt`, `createdBy`, or `updatedBy`.
- The engine must work locally, in CI, and in a basic server environment with only a supported Node.js release and Git.
- The current repository state must remain useful without a network connection.
- Data files are authoritative. Rendered pages, indexes, caches, and reports are derived output.
- Never fetch external references automatically. A user may open or import one explicitly.
- Keep the model generic. Organization-specific fields belong in namespaced extensions.
- Prefer explicit, inspectable behavior over automation that changes audit records without review.
- UI, HTTP, and CLI workflows must call the same domain functions so headless agents receive the same calculations, validation, and output as browser users.

## Package constraints

### `filegrc`

- Use Node.js built-in modules only.
- Do not add runtime, development, test, build, or rendering dependencies.
- Do not require a compilation or bundling step.
- Use `node:test` and other built-in tooling for tests.
- Keep file parsing, validation, Git access, HTTP handling, and rendering behind small internal interfaces.
- CRUD operations may write files but must not create commits unless the user explicitly requests it.
- Writes must be atomic and must reject paths outside the workspace.

### `create-filegrc`

- Follow the `npx create-*` convention and support `npx create-filegrc@latest`.
- Generate a private Node.js project whose only package dependency is `filegrc`.
- Resolve the current `filegrc` release when generating, write a normal semver range, and create a lockfile. Do not write the literal dependency specifier `latest`.
- Do not overwrite a non-empty target without explicit user approval.
- Initialize Git when the target is not already inside a Git worktree.
- Keep the template usable without private services or organization-specific values.
- Define create-time prompts once in `packages/create-filegrc/template-parameters.json`.
- Keep create-time prompts limited to values needed throughout the initial repository. Prefer documented defaults and later edits for optional configuration.
- Replace every template token during creation and fail if a token is unknown or remains unresolved.

## Data rules

The authoritative model registry is `packages/filegrc/model/v1.json`. Model v1 is published. Before changing it, decide whether existing v1 workspaces remain valid. Keep compatible v1 changes, starter data, generated docs, and tests in sync. Add a new model version and an explicit migration path when a change would make existing v1 workspaces invalid.

- Use UTF-8 JSON for structured records and Markdown for long-form content.
- Store canonical long-form Markdown beside its structured JSON record. filegrc derives companion names from the JSON location and Markdown slot; do not store those paths in record data.
- Structure fields only when the engine needs them for validation, filtering, relationships, lifecycle rules, due-date calculations, or audit-period completeness.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sunpeak-AI/filegrc](https://github.com/Sunpeak-AI/filegrc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
