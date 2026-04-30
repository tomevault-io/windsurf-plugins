---
trigger: always_on
description: - This repository is a wrapper for the Codex Autoresearch plugin. The active product code lives under `plugins/codex-autoresearch`.
---

# AGENTS.md

## Project Shape

- This repository is a wrapper for the Codex Autoresearch plugin. The active product code lives under `plugins/codex-autoresearch`.
- Treat `plugins/codex-autoresearch` as the package root for implementation, checks, tests, release metadata, the main plugin skill, MCP server code, and dashboard assets.
- The root `README.md` is the only README and the public-facing documentation surface. Keep root-relative links valid.
- The root `CHANGELOG.md` is the release-note surface. Update it for user-facing behavior, docs, skill, command-surface, dashboard, MCP, migration, or version changes.
- Do not assume root-level npm scripts exist. The package scripts live in `plugins/codex-autoresearch/package.json`.

## Product Intent

- Codex Autoresearch is a measured, resumable optimization loop for Codex: choose one metric, run a benchmark, keep or discard changes with evidence, preserve ASI, export the dashboard, and finalize kept work into reviewable branches.
- Preserve the core contract: benchmark commands print `METRIC name=value`; primary metrics drive decisions; secondary metrics explain tradeoffs.
- Prefer durable loop state over chat memory. Important session knowledge belongs in `autoresearch.md`, `autoresearch.ideas.md`, `autoresearch.jsonl`, `autoresearch.research/<slug>/`, dashboard snapshots, and ASI.
- For qualitative or broad product-study work, convert accepted findings into a `quality_gap` checklist. `quality_gap=0` closes the accepted checklist for the current round; it does not mean discovery is permanently complete.
- The main product gap from prior work is guided decision flow. When product direction is ambiguous, prioritize making the next safe action obvious over adding more raw panels or low-level knobs.

## Local Plugin Routing

- When this repo is the target, use the repo-local plugin over any globally installed or marketplace-cache copy.
- From the repository root, prefer:

```bash
node plugins/codex-autoresearch/scripts/autoresearch.mjs mcp-smoke
node plugins/codex-autoresearch/scripts/autoresearch.mjs doctor --cwd plugins/codex-autoresearch --check-benchmark
node plugins/codex-autoresearch/scripts/autoresearch.mjs next --cwd plugins/codex-autoresearch
node plugins/codex-autoresearch/scripts/autoresearch.mjs export --cwd plugins/codex-autoresearch
```

- From `plugins/codex-autoresearch`, use `node scripts/autoresearch.mjs ...` and pass the target project with `--cwd`.
- The canonical agent workflow doc is `plugins/codex-autoresearch/skills/codex-autoresearch/SKILL.md`. Update it when command flow, safety rules, dashboard reporting behavior, or finalization behavior changes.

## Implementation Map

- CLI command handling is split between `scripts/autoresearch.mjs` and `lib/cli-handlers.mjs`.
- MCP tool schemas and argument validation live in `lib/mcp-tool-schemas.mjs`; in-process dispatch lives in `lib/mcp-interface.mjs`; stdio CLI bridging lives in `lib/mcp-cli-adapter.mjs`; the lightweight wrapper lives in `scripts/autoresearch-mcp.mjs`.
- Session state and metrics logic live in `lib/session-core.mjs`; keep script-facing and library-facing behavior in sync.
- Dashboard data shaping lives in `lib/dashboard-view-model.mjs`; dashboard HTML/CSS/JS lives in `assets/template.html`.
- Runner behavior lives in `lib/runner.mjs`; recipes and research-gap logic live in `lib/recipes.mjs` and `lib/research-gaps.mjs`.
- Finalization behavior lives in `scripts/finalize-autoresearch.mjs` and related tests.
- Product-quality expectations are encoded in `scripts/perfection-benchmark.mjs`. Update this benchmark when public commands, docs, MCP contracts, or session hygiene expectations intentionally change.
- Keep `skills/codex-autoresearch/SKILL.md`, the root README, the root changelog, tests, and MCP schemas synchronized when user-facing contracts change.

## MCP And Cache Drift

- The local MCP config is `plugins/codex-autoresearch/.mcp.json`. It should launch `node ./scripts/autoresearch-mcp.mjs` with `cwd: "."` and a startup timeout.
- Keep `scripts/autoresearch-mcp.mjs` lightweight. It should answer `initialize` and `tools/list` from static metadata before loading or spawning heavier CLI behavior for `tools/call`.
- For MCP startup failures, do not guess. Check the direct smoke path first:

```bash
node plugins/codex-autoresearch/scripts/autoresearch.mjs mcp-smoke
```

- If the installed Codex plugin behaves differently from source, verify the live runtime with `codex mcp get codex-autoresearch` and inspect the versioned cache under the user's Codex plugin cache. Source files and installed runtime can drift.
- Typical MCP failure layers are wrong cwd, stale marketplace cache, old versioned cache, schema/tool metadata mismatch, startup noise, and slow full-CLI imports. Identify the layer before changing timeouts.

## Autoresearch Workflow Safety

- Before starting or resuming a loop, check Git state. If the tree is dirty, distinguish user edits from experiment edits and avoid broad cleanup.
- After `next`, log the packet with `--from-last` instead of retyping parsed metrics. Choose `keep`, `discard`, `crash`, or `checks_failed` deliberately.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheGreenCedar/codex-autoresearch](https://github.com/TheGreenCedar/codex-autoresearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
