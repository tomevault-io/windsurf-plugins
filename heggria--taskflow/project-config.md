---
trigger: always_on
description: > Instructions for AI coding agents working on taskflow.
---

# AGENTS.md

> Instructions for AI coding agents working on taskflow.

## Project Overview

taskflow is a **declarative DAG orchestration runtime** for coding agents — it runs on the [Pi coding agent](https://pi.dev), on [OpenAI Codex](https://github.com/openai/codex), on [Claude Code](https://claude.com/product/claude-code), on [OpenCode](https://opencode.ai), and on [Grok Build](https://docs.x.ai/build/overview). It lets users define multi-phase workflows (fan-out, gate, loop, tournament, approval, sub-flow composition) as JSON DSL, executes them via isolated subagent processes, and returns only the final result — intermediate transcripts never enter the host context window.

**Language:** TypeScript (ES2022, ESM, `--experimental-strip-types` for direct execution in dev)\
**Runtime:** Node.js ≥ 22.19 (uses `fs.globSync`, `Atomics.wait`)\
**Dependencies:** Zero runtime deps. The private `charterarc` experiment depends only on `taskflow-core`. The Pi adapter (`pi-taskflow`) peer-depends on `@earendil-works/pi-{agent-core,ai,coding-agent,tui}`; the host-neutral MCP server (`taskflow-mcp-core`) and the four MCP host adapters (`codex-taskflow`, `claude-taskflow`, `opencode-taskflow`, `grok-taskflow`) all depend on `taskflow-core` (the adapters also depend on `taskflow-mcp-core`). Everything depends on `typebox`.\
**Layout:** pnpm-workspace monorepo — `taskflow-core` (engine), private `charterarc` (project template loop), `taskflow-mcp-core` (MCP + DAG SVG), `taskflow-hosts` (codex/claude/opencode/grok runners), **`taskflow-dsl`** (S4: `.tf.ts` → Taskflow → FlowIR; CLI `taskflow-dsl`), `pi-taskflow`, `codex-taskflow`, `claude-taskflow`, `opencode-taskflow`, `grok-taskflow` (host delivery packages).\
**Build:** each package compiles to `dist/*.js` + `.d.ts` (`tsc`); published packages ship `dist` (Node refuses to type-strip `.ts` under `node_modules`). Dev resolves the TypeScript sources directly via a `development` export condition — no build needed to typecheck or test.

## Architecture

```
packages/
├─ taskflow-core/          ← host-neutral engine (zero host-SDK deps; only typebox)
│  ├─ src/
│  │  ├─ index.ts          ← barrel: re-exports the engine's public surface
│  │  ├─ schema.ts         ← Taskflow DSL TypeBox schema, validation, desugar, topo sort
│  │  ├─ runtime.ts        ← orchestration engine: DAG resolution, phase execution, caching
│  │  ├─ runner-core.ts    ← host-neutral helpers: failure classification, NDJSON accumulator,
│  │  │                       sanitize, mapWithConcurrencyLimit (the pure half of the old runner)
│  │  ├─ interpolate.ts    ← template interpolation ({steps.X.output}), condition parser (when/eval)
│  │  ├─ agents.ts         ← agent discovery (~/.pi/agent/agents/*.md + .pi/agents/*.md)
│  │  ├─ store.ts          ← persistence: flow definitions + run state + file locks + index
│  │  ├─ cache.ts          ← cross-run memoization: fingerprint resolution + CacheStore
│  │  ├─ verify.ts         ← static DAG verification (zero-token structural analysis)
│  │  ├─ compile.ts        ← Mermaid diagram + verify report renderer
│  │  ├─ context-store.ts  ← Shared Context Tree: blackboard + supervision (ctx_read/write/report/spawn)
│  │  ├─ detached-runner.ts← spawn-only entry for background runs (NOT in the barrel)
│  │  ├─ usage.ts          ← token/cost accounting (UsageStats type + aggregation)
│  │  ├─ stale.ts / workspace.ts / flowir/  ← staleness, worktrees, FlowIR compile seam
│  │  │                                       (S0: compileTaskflowToFlowIR + hashFlowIR → ir:<64-hex>)
│  │  ├─ exec/             ← event log schema + fold + S2 kernel (step/driver; default OFF)
│  │  ├─ replay.ts         ← offline what-if replayRun (zero tokens; no runtime/driver import)
│  │  ├─ resume.ts          ← fork/apply/validate resume overrides + transitiveDownstream (issue 5)
│  │  ├─ final-output.ts    ← shared resolveFinalOutput: final-phase selection + output source attribution (issue 6)
│  │  ├─ build-info.ts      ← getBuildInfo(): packageVersion/gitCommit/schemaVersion (build-time stamp; issue 4)
│  │  ├─ trace.ts          ← TraceEvent / FileTraceSink / readTrace
│  │  ├─ host/runner-types.ts ← the host-neutral SubagentRunner contract + vendored CoreMessage
│  │  ├─ runner-core.ts  ← ALSO hosts the shared `runSubagentProcess` (spawn/idle/abort/classify) +
│  │  │                     `SubagentAccumulator` + `unknownAgentResult` reused by every host runner
│  │  ├─ typebox-helpers.ts / frontmatter.ts / paths.ts  ← vendored pi-SDK helpers (zero-dep)
│  │  └─ agents/           ← 18 built-in agent definitions (*.md with YAML frontmatter; copied to dist)
│  └─ test/              ← engine unit tests
├─ charterarc/            ← private vNext experiment: observe → one ordinary Taskflow → re-observe
│  ├─ src/project/        ← defineProject + the minimal project cycle and public types
│  └─ test/               ← project-cycle, package-boundary, and real phase-docs tests
├─ taskflow-mcp-core/           ← host-neutral MCP server (depends on taskflow-core)
│  ├─ src/mcp/            ← jsonrpc.ts (stdio JSON-RPC), server.ts (taskflow_* tools; parameterized by
│  │                        a SubagentRunner), svg.ts (DAG SVG/outline renderer)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heggria/taskflow](https://github.com/heggria/taskflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
