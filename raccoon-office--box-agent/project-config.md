---
trigger: always_on
description: `box_agent/` contains the application code: `agent.py` drives the execution loop, `cli.py` exposes the CLI, `llm/` wraps model providers, `tools/` holds built-in tools, `acp/` hosts the ACP server, and `config/` stores example config files. `tests/` contains the automated test suite, with files such as `test_agent.py` and `test_mcp.py`. `examples/` provides runnable demos, while `docs/` and `docs/assets/` hold contributor-facing documentation and images. Treat `workspace/` as runtime scratch spa
---

# Repository Guidelines

## Project Structure & Module Organization

`box_agent/` contains the application code: `agent.py` drives the execution loop, `cli.py` exposes the CLI, `llm/` wraps model providers, `tools/` holds built-in tools, `acp/` hosts the ACP server, and `config/` stores example config files. `tests/` contains the automated test suite, with files such as `test_agent.py` and `test_mcp.py`. `examples/` provides runnable demos, while `docs/` and `docs/assets/` hold contributor-facing documentation and images. Treat `workspace/` as runtime scratch space, not committed source.

## Code Discovery & Understand Anything

For architecture, ownership, dependency, onboarding, and change-impact questions,
use this default order:

1. Check the committed `.understand-anything/knowledge-graph.json` and read its
   `project.gitCommitHash` as the analyzed source baseline. Compare that
   baseline with later source changes before treating the graph as current.
   Cross-check `.understand-anything/meta.json` for the refresh timestamp,
   baseline commit, graph version, and analyzed file count. If present, inspect
   `.understand-anything/last-run-summary.json` for refresh status.
2. Use `.understand-anything/knowledge-graph.json`, or the most relevant
   `.understand-anything/domain-graphs/*knowledge-graph.json` when such a graph
   exists, as the initial codebase index.
3. Extract only the relevant nodes, edges, layers, or tour steps with `jq`,
   `rg`, or a focused keyword search. Do not load or summarize the entire graph
   when a narrow query is sufficient.
4. Open the smallest useful set of real source files and verify the graph-based
   conclusion with direct reads, `rg`, focused tests, logs, or runtime probes as
   appropriate.

Treat every graph as a navigation index, not the source of truth. If the graph,
metadata, or tooling is missing or stale, state the limitation and continue
with normal source search when the task can still proceed; recommend a refresh
when it would materially improve the result.

Keep the shared `.understand-anything/knowledge-graph.json`, `meta.json`,
`fingerprints.json`, `.understandignore`, and `config.json` in Git. The graph,
metadata, and fingerprints form one refresh baseline and must be regenerated
and reviewed together; do not hand-edit them. Keep `last-run-summary.json`,
intermediate files, dashboard tokens, trash, and caches local. Add any future
shared domain graph to Git intentionally together with its scope documentation.

## AI Development Workflow

Automated coding assistants must treat this section as the default execution
protocol for repository work.

### Before Editing

1. Read the user request literally and keep inspect, analyze, review, and status
   requests read-only. Do not edit, commit, merge, package, install, or publish
   unless the user asked for that action.
2. Run `git status --short --branch` before changing files. Preserve unrelated
   tracked, staged, and untracked work. Do not reset, clean, stash, switch the
   user's branch, or rewrite existing changes to make the task easier.
3. Classify the behavior before choosing files:
   - host/protocol translation belongs in `box_agent/acp/` or `box_agent/cli.py`
   - tools, skills, providers, storage, and reusable workflow policy belong in
     their capability modules
   - shared contracts and loop invariants belong in the stable API/kernel layer
4. Inspect the nearest existing implementation and its tests before adding a
   new helper, abstraction, directory, dependency, or configuration surface.
5. Define the observable success condition and the smallest check that proves
   it. Distinguish requirements from assumptions and state any assumption that
   materially affects behavior.

### While Editing

- Make the smallest reviewable change that solves the requested behavior.
  Avoid unrelated refactors, formatting churn, renames, and speculative
  compatibility layers.
- Implement shared behavior once behind a shared contract. Keep CLI and ACP as
  adapters; do not maintain parallel copies of the same runtime policy.
- Do not modify `core.py` or another stable-kernel file for product-specific
  behavior that can be expressed through a Tool, Skill, Hook, event consumer,
  run option, completion gate, or `WorkflowPolicy`.
- Preserve public API compatibility unless the task explicitly changes the
  contract. Prefer additive event and option changes; document migrations for
  removals or semantic changes.
- Add or update a direct regression test for every behavior change. Test names
  should describe observable outcomes rather than internal implementation.
- Do not hand-edit generated manifests, lock state, timestamps, or packaged
  artifacts when a repository generator or installer owns them.
- Never put secrets, tokens, local config, user data, logs, `workspace/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Raccoon-Office/Box-Agent](https://github.com/Raccoon-Office/Box-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
