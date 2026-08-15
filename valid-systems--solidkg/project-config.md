---
trigger: always_on
description: This file provides guidance to Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Agents when working with code in this repository.

## Project Overview

SolidKG is a local-first code intelligence library + CLI + MCP server. It parses any supported codebase with tree-sitter, stores symbols, pairwise edges, semantic hyperedges, and files in SQLite (FTS5), and exposes a knowledge graph to AI agents (Claude Code, Cursor, Codex CLI, opencode, Hermes Agent) over MCP. Per-project data lives in `.solidkg/`. Extraction is deterministic — derived from AST, not LLM-summarized.

Distributed as `solidkg` on npm; same binary serves as installer, indexer, and MCP server.

## Build, Test, Run

Use pnpm for local agent work in this repo. The package scripts are npm-compatible for published users, but do not use bare `npm` while developing here unless a release workflow explicitly requires it.

```bash
pnpm exec tsc           # TypeScript build/check
pnpm run copy-assets    # copy schema.sql and native-grammars.json into dist/
node -e "require('fs').chmodSync('dist/bin/solidkg.js', 0o755)"  # executable bit

pnpm run dev            # tsc --watch
pnpm run clean          # rm -rf dist

pnpm test               # vitest run (all)
pnpm run test:watch

# Single test file / pattern
pnpm exec vitest run __tests__/installer-targets.test.ts
pnpm exec vitest run __tests__/extraction.test.ts -t "TypeScript"
```

`copy-assets` (called from `build`) copies `src/db/schema.sql` and `src/extraction/native-grammars.json` into `dist/`. `prepare:native-runtime` stages the host parser libraries in `.native-grammars/<platform>-<arch>`, and `build-bundle.sh` copies that allowlisted set into the platform bundle.

Node engines: `>=24.0.0`; use Node 24 LTS for local development when possible. The CLI hard-blocks Node <24 unless explicitly overridden (see `src/bin/node-version-check.ts`). Released bundles ship their own Node runtime.

## Public source checkout, repair, and MCP adoption

Use this workflow when an agent is operating in a clone or GitHub source archive on a user's computer. The goal is to prove that the downloaded source is complete before changing any user-level coding-tool configuration.

### Install and prove the checkout

1. Confirm the checkout is running Node 24 and the exact pnpm version in `package.json#packageManager`. If pnpm is unavailable, offer to enable it with Corepack; do not change the user's global toolchain without permission.
2. Install exactly the committed dependency graph and run the complete deterministic readiness suite:

   ```bash
   pnpm install --frozen-lockfile
   pnpm run verify:source:full
   ```

   `verify:source:full` builds from TypeScript, verifies copied SQL/native grammar metadata and the staged host parser libraries, requires every maintained vendored SCIP source tree, indexes a fresh two-file project with real SQLite and tree-sitter, generates/imports/links a real TypeScript SCIP index, exercises library search, starts the real stdio MCP server, calls `solidkg_explore`, checks agent-install diagnostics are read-only, runs every Vitest file present in the checkout, and validates release-package file lists without publishing. The full internal repository contains additional maintainer-only benchmark/evidence contracts that are absent from the initial public archive.
3. The top-level `vendored/` directory is part of the public source contract. It contains pinned SolidKG compatibility variants of the SCIP indexers, including SolidKG-specific changes where official upstream behavior diverges from the integration contract. Do not replace them with upstream HEAD or an arbitrary official binary when reproducing or repairing SolidKG behavior. `pnpm run build` does not compile every polyglot indexer: when the user needs a non-bundled SCIP language, use the matching `vendored/scip-*` README/toolchain, expose the command expected by `src/scip/index.ts` on `PATH`, run `solidkg scip generate --path <project>`, and confirm the generated/imported/linked state with `solidkg status <project>`.

Both `vendored/` and `src/extraction/native-grammars.json` must be present in a public source download. The former preserves the supported indexer implementations; the latter is the allowlist used to stage parser libraries from the pinned language-pack release. Never "repair" either with a floating upstream revision, an empty placeholder, or an arbitrary grammar/indexer binary.

### Correct a failed source install

- Wrong Node major: switch to Node 24, remove only this checkout's `node_modules`, rerun `pnpm install --frozen-lockfile`, then rerun the verifier.
- Wrong pnpm version: use the version pinned by `packageManager`; do not weaken or remove `--frozen-lockfile`.
- Lockfile mismatch: stop and inspect the manifest/lock diff. A maintainer must deliberately refresh the appropriate lockfile; do not silently regenerate dependency resolutions just to obtain a pass.
- Missing `src/db/schema.sql` or `src/extraction/native-grammars.json`: treat the download as incomplete and re-clone or re-download the same commit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Valid-Systems/SolidKG](https://github.com/Valid-Systems/SolidKG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
