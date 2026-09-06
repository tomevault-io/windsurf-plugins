---
trigger: always_on
description: A standalone, provider-agnostic autonomous agent loop for JavaScript. Built on the Vercel AI SDK. Zero internal dependencies beyond `ai` and `zod`.
---

# agent-do — Development Guide

## What This Is

A standalone, provider-agnostic autonomous agent loop for JavaScript. Built on the Vercel AI SDK. Zero internal dependencies beyond `ai` and `zod`.

## Project Structure

```
src/
  agent.ts            — createAgent() — the main entry point
  loop.ts             — runAgentLoop / streamAgentLoop — core loop implementation
  loop-history.ts     — conversation-history hygiene for the loop (#33)
  types.ts            — all TypeScript interfaces and types
  index.ts            — all exports

  stores.ts           — MemoryStore + FileEntry + SearchOptions interfaces
  stores/
    in-memory.ts      — InMemoryMemoryStore (testing/prototyping)
    filesystem.ts     — FilesystemMemoryStore (Node.js persistent)
    sandbox.ts        — SandboxBackedMemoryStore (adapts a SandboxApi)
    file-lock.ts      — opt-in cross-process file locking (#15 Tier 1)
    agent-id.ts       — shared agentId validation
    search-matcher.ts — shared line matcher for store search()

  tools/
    file-tools.ts     — createFileTools() — internal primitive (not exported)
    workspace-tools.ts — createWorkspaceTools() — project files + deny list
    memory-tools.ts   — createMemoryTools() — the agent's private scratchpad
    shell-tool.ts     — createShellTool() — SandboxApi.exec as one tool
    deny-list.ts      — path-scoped access policy for file tools
    content-guards.ts — guards applied before tool data flows back to the model
    types.ts          — ToolResult (structured tool results, #48)

  sandbox/
    types.ts          — SandboxApi contract (#3)
    connectors/       — host passthrough + vercel-labs/just-bash

  skills.ts           — skill system (parse, build prompt, InMemorySkillStore)
  policies.ts         — typed system-prompt modules (#80)
  routines.ts         — named prompt-as-macro procedures (#77)
  slash-commands.ts   — deterministic pre-model dispatch (#76)
  scheduled-tasks.ts  — cron-driven agent runs with lock-file concurrency (#79)
  mcp.ts              — mount external MCP tool servers
  permissions.ts      — permission evaluation logic
  usage.ts            — UsageTracker + cost estimation + DEFAULT_PRICING
  orchestrator.ts     — multi-agent orchestration (master + workers)
  debug-middleware.ts — debug/observability middleware (#72)

  prompts/            — composable system prompts (builder, sections, templates)
  testing/
    index.ts          — createMockModel() for testing
  eval/
    index.ts          — eval framework exports
    types.ts          — eval types (assertions, cases, results)
    assertions.ts     — assertion evaluators (13 types)
    runner.ts         — eval runner (defineEval, runEvals)

  cli.ts              — CLI entry point (npx agent-do)
  cli/
    args.ts           — argument parser + stdin reader
    prompt.ts         — prompt mode (one-shot + interactive)
    script.ts         — script mode (npx agent-do run)
    eval-cmd.ts       — eval mode (npx agent-do eval)
    shellm.ts         — prompt files as executable scripts (#16)
    scheduled-tasks-cmd.ts — `agent-do scheduled-tasks` (#79)
    agents.ts         — create / list / load saved agent configs
    resolve-model.ts  — dynamic provider/model resolution + DEFAULT_MODELS
    provider-tools.ts — `--provider-tool` registry (provider-native tools)
    permission-handler.ts — interactive permission prompts (#17)
    debug-config.ts   — `--log-level` → AgentConfig.debug (#72)
    render.ts         — shared ProgressEvent rendering
    warnings.ts       — startup "no sandbox" warning

tests/              — vitest unit tests (one file per module)
examples/           — focused single-feature examples (npx tsx examples/NN-name.ts)
evals/              — eval suites: behaviour.ts (mock tier, CI) + quality.live.ts
docs/               — api-reference.md, sandbox.md, supply-chain.md, guides/
demos/              — comprehensive end-to-end demo applications
  assistant/        — interactive CLI assistant with persistent memory
  research-team/    — multi-agent research pipeline (master + workers)
  code-reviewer/    — automated code review (read-only filesystem)
  chief-of-staff/   — multi-agent scheduling / triage assistant
  engineering-team/ — multi-agent engineering workflow
```

Keep this tree accurate — it is the first thing an agent reads before
touching the repo, and a stale map costs more than it saves.

## Rules for Every Change

1. **Tests first** — write or update tests for every change. Run `npm test` before committing.
2. **Examples** — if the change affects user-facing API, update the relevant example in `examples/`.
3. **Demos** — if the change affects core API, verify all demos in `demos/` still work. Demos use `"agent-do": "file:../../"` so they always use the local version, which makes them the only check that exercises the *published export surface* the way a real consumer does. CI runs a credential-free smoke (`Demo smoke (no credentials)`): every demo must reach its "No provider API key found" check. **If you remove or rename an export, grep `demos/` before you assume nothing uses it** — every demo silently broke on `createFileTools` for months because nothing ran them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulKinlan/agent-do](https://github.com/PaulKinlan/agent-do) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
