---
trigger: always_on
description: - Product overview: `README.md` and `README.zh-CN.md`.
---

# OKF Harness Agent Instructions

## Source of Truth

- Product overview: `README.md` and `README.zh-CN.md`.
- User workflows: `docs/WORKFLOWS.md` and `docs/zh-CN/WORKFLOWS.md`.
- CLI contract: `docs/CLI.md` and `docs/zh-CN/CLI.md`.
- Product terminology: `CONTEXT.md`.
- Architecture decisions: `docs/adr/`.
- Public roadmap: `docs/ROADMAP.md` and `docs/zh-CN/ROADMAP.md`.
- Current public scope: CLI doctor/init/status/check/source/ingest/search/evidence/read/graph/agent install/bootstrap, retired lint compatibility, Claude/Codex adapter rendering, docs, npm metadata, CI tarball smoke, and one small example workspace.

## Project Boundaries

- OKF Harness is agent-first, local-first, terminal-native, and open source.
- Runtime target: Node.js 22+, TypeScript, ESM, pnpm.
- Current public scope supports Claude Code and Codex first. Pi and OpenCode are roadmap work.
- Out of scope for the current public scope: Obsidian runtime code, GUI, cloud sync, accounts, team permissions, vector databases, background daemons, automatic web crawling, silent bulk wiki rewrites, and private agent runtime.
- The root package stays private. Publish only the scoped core, agent-pack, and CLI packages.
- The core package must not depend on CLI, Agent, or other higher-level packages.
- The agent-pack package renders shared templates for Claude and Codex adapters; do not maintain divergent manual skill copies.
- The cli package connects core and agent-pack behavior through `okfh --json`.

## Repository Map

- `packages/core`: OKF parsing, config, manifest, path safety, lint, search, graph, source handling.
- `packages/cli`: `okfh` command line entrypoint and commands.
- `packages/agent-pack`: Claude/Codex adapter renderers and shared skill templates.
- `packages/core/src/workspace/index.ts`: generated OKF Harness workspace skeleton and workspace plan until a durable template directory exists.
- `packages/core/test/fixtures/valid-workspace`: fixture workspace and sample sources used by core and CLI tests.

## Hotspot Ownership

- `README.md`, `docs/WORKFLOWS.md`, `docs/CLI.md`, `docs/ROADMAP.md`, `CONTEXT.md`, and `docs/adr/` own the public product surface. Keep user-facing docs concise and avoid exposing internal planning documents.
- `packages/core/src/workspace/index.ts` owns the OKF Harness workspace skeleton and workspace plan. Keep it free of agent-pack dependencies. Verify with `pnpm test packages/core/test` and `pnpm typecheck`.
- `packages/core/src/evidence/index.ts` and `packages/core/src/read/index.ts` own bounded answer inputs and continuation reads. Keep normal answer workflows on synthesized `wiki/` content, preserve JSON limit/citation metadata, and verify with `pnpm test packages/core/test/evidence.test.ts packages/core/test/read.test.ts` plus `pnpm typecheck`.
- `packages/agent-pack/src/index.ts` owns Claude/Codex skill rendering, root guidance managed blocks, and adapter install planning. Verify with `pnpm test packages/agent-pack/test` and `pnpm typecheck`.
- Agent skill or reference changes must use the `writing-great-skills` skill before editing `packages/agent-pack/templates/okf-harness/` or generated skill guidance.
- `packages/cli/src/index.ts` owns the terminal-native command registration and connects core with agent-pack. Keep CLI output compatible with `okfh --json` and avoid alternate default tool channels. Keep rendering, option parsing, and error normalization in dedicated CLI modules when they grow beyond command wiring. Verify with `pnpm test packages/cli/test` and `pnpm typecheck`.
- `packages/cli/test/bootstrap.test.ts` owns global bootstrap and adapter install contract coverage. Keep shared temp-workspace and JSON helpers in `packages/cli/test/helpers.ts`; split command-domain tests instead of growing one catch-all file. Verify with `pnpm test packages/cli/test`.
- `pnpm-lock.yaml` owns dependency resolution state only. Do not hand-edit it; update it through pnpm when package manifests change, then verify with `pnpm install --frozen-lockfile` or the normal CI command set.

## Agent skills

Engineering workflow skills share repo-local setup through `docs/agents/` so issue, triage, and domain assumptions stay versioned with the project.

### Issue tracker

Use GitHub Issues for issues and PRDs. External PRs are not a triage request surface. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the default triage state strings: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, and `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Use a single-context domain documentation layout. See `docs/agents/domain.md`.

## Working Rules

- Before editing, check the current tree with `git status --short`.
- Keep changes scoped to the current public scope or the user's explicit request.
- Do not auto-commit, tag, push, publish, or create releases unless the user explicitly asks in the current turn.
- Do not store API keys, tokens, or credentials in tracked files.
- Do not edit generated raw source files in OKF Harness workspaces. `raw/sources/` is immutable by design.
- Do not edit ignored `dist/` output by hand. Build it from source when package contents need verification.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pumblus/okf-harness](https://github.com/pumblus/okf-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
