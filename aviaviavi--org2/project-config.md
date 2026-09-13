---
trigger: always_on
description: This repository contains two closely related layers:
---

# OpenOrg / Org2 Agent Guide

This repository contains two closely related layers:

- **OpenOrg** is the user-facing macOS and iOS workspace.
- **Org2** is the independently specified compiler/runtime and semantic profile
  for ordinary `.org` documents, plus the CLI, schemas, publishing system,
  plugin runtime, and editor tooling beneath OpenOrg. Existing `.org2` files
  and structured runtime records remain supported for compatibility.

These instructions are runtime-neutral. OpenClaw, Codex, Claude Code, and other
agents may work here, but no agent runtime, app cache, generated index, or model
memory is the source of truth.

## Start here

1. Read this file, inspect `git status --short --branch`, and check the current
   worktree before editing. Preserve unrelated work in a dirty checkout.
2. Decide whether the request changes this repository or an Org2 corpus. This
   file governs repository work. Corpus work also follows the nearest corpus
   `AGENTS.md` and `org2.json`; ordinary `.org2` and `.org` files remain
   canonical there.
3. In a fresh checkout, install the locked dependencies with `npm ci`. Reuse
   an existing `node_modules/` only when the lockfile has not changed.
4. Build before using the checkout's CLI, then ask the implementation what it
   supports:

   ```sh
   npm run build
   npm run org2 -- agent capabilities
   npm run org2 -- --help
   npm run org2 -- COMMAND --help
   ```

   Prefer `npm run org2 -- ...` or `node dist/cli.js ...` during development.
   A globally installed `org2` may expose a different version.
5. Choose the smallest affected surface and its focused tests before making a
   broad change. Check documentation impact at the same time as code impact.

## Source of truth and safety

- Treat inspectable plain text as canonical. Derived indexes, compiled context,
  reports, app state, generated pages, and runtime state must be disposable or
  reconstructible.
- Preserve source ranges, stable IDs, file/line citations, provenance, hashes,
  and review state across compiler and agent workflows.
- Most Org2 mutations preview by default and require `--apply`. Inspect the
  preview or JSON envelope before applying it.
- Keep generated or uncertain corpus work in reviewable zones such as `views/`
  or `compiled/`; promotion into canonical `notes/` is an explicit action.
  Keep immutable imports and provider payloads in `raw/`.
- Never put credentials, tokens, cookies, private keys, model credentials, or
  machine-local bindings in source, fixtures, corpora, runs, plugins, generated
  artifacts, or documentation examples.
- Multi-corpus reads require explicit mounts. A corpus visible in OpenOrg is not
  implicit agent authority, and writes remain scoped to one active corpus.
- Do not perform external side effects such as sending messages, creating
  tickets, publishing, installing plugins, restarting services, or changing a
  user's daily app unless the user explicitly authorizes that action.
- Durable run and workflow writes are guarded and atomic. Carry
  `--if-revision` when state crosses requests; never hand-edit machine-state
  blocks in `.org2/runs/*.org2`.

## Working in a concurrent repository

- Existing modifications and untracked files belong to the user or another
  worker unless you know otherwise. Never discard, stage, format, or commit them
  as a side effect of your task.
- If the primary checkout is dirty or behind `origin/main`, use a clean,
  isolated worktree based on the current remote tip. Fetch and rebase again
  immediately before pushing because this repository changes frequently.
- Keep commits single-purpose. Do not mix generated output, release metadata, or
  another worker's feature into a convenient commit.
- Do not commit, push, tag, publish, or open a pull request unless the user
  requested that external action. A request to push a scoped change authorizes
  the normal commit and push needed for that change, not unrelated work.
- Avoid destructive Git commands. Resolve exact targets first, and prefer a
  recoverable or isolated workflow when a checkout contains work in progress.

## Product and architecture

- The TypeScript compiler/runtime under `src/` owns parsing, semantics, source
  ranges, IDs, links, agenda behavior, corpus operations, agent interfaces,
  publishing, plugin contracts, and LSP behavior. Build output goes to `dist/`.
- Apps and editor integrations consume shared semantics. Do not create a second
  parser, a private canonical database, or app-only language behavior.
- `apps/macos/Org2Workspace/` is the native OpenOrg workspace. It supports
  local and explicitly mounted corpora, reading/editing, capture, meetings,
  publishing, data views, plugins, durable agent work, and per-thread AI chat
  through configured runtimes such as OpenClaw and Codex.
- `apps/ios/Org2Mobile/` is the source-distributed mobile capture, corpus, chat,
  and approval client. Its inbox flows are transport boundaries, not new
  language semantics.
- `integrations/openclaw/` contains the optional `org2-lifecycle` adapter.
  OpenClaw is a native integration, not the owner of runs, workflows, approvals,
  or corpus state.
- The plugin runtime installs content-addressed Git packages, pins exact commits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aviaviavi/org2](https://github.com/aviaviavi/org2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
