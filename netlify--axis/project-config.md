---
trigger: always_on
description: AXIS (Agent Experience Index Score) is a synthetic testing framework for AI agents. It runs agents against scenarios, captures transcripts, and produces graded scores across four dimensions: goal achievement, environment quality, service quality, and agent quality.
---

# AGENTS.md

## Project Overview

AXIS (Agent Experience Index Score) is a synthetic testing framework for AI agents. It runs agents against scenarios, captures transcripts, and produces graded scores across four dimensions: goal achievement, environment quality, service quality, and agent quality.

- ESM TypeScript, built with `tsc`, tested with `vitest`, CLI via `commander`
- Live terminal display uses `ink` (React for CLIs), rendered to stderr
- Runner is fully decoupled from display via a `Logger` interface

## Terminology

- **AXIS Result** (not "AXIS Score") -the composite 0–100 number. "AXIS Score" reads as "score score" since AXIS already stands for "Agent Experience Index **Score**".
- Use "AXIS Result" in all user-facing text, display output, and documentation.
- The internal property names (`axisScore`, `averageAxisScore`) are fine as code identifiers.

## Architecture

| Layer    | Key Files                                                  | Purpose                                                                                                                                                                                                                                                        |
| -------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CLI      | `src/cli.ts`                                               | Entry point, ink display, signal handling                                                                                                                                                                                                                      |
| Runner   | `src/runner/runner.ts`, `lifecycle.ts`                     | Job orchestration, concurrency, isolation; `runLifecyclePhase` captures `$AXIS_OUTPUT` markdown for `setup`/`teardown`/`beforeAll`/`afterAll` phases                                                                                                           |
| Adapters | `src/adapters/*.ts`                                        | Spawn agent CLIs, parse NDJSON streams                                                                                                                                                                                                                         |
| Scoring  | `src/scoring/`                                             | LLM judge + interaction-based evaluation pipeline                                                                                                                                                                                                              |
| Reports  | `src/reports/writer.ts`, `reader.ts`                       | Persistent `.axis/reports/` store                                                                                                                                                                                                                              |
| Config   | `src/config/loader.ts`, `remote-scenarios.ts`              | Parses `axis.config.*`; `mergeRemoteConfig` clones any git-URL entries in `scenarios` into `.axis/remotes/`, inlines their paths, AND folds each remote repo's `env`/`mcp_servers`/`skills`/`artifacts`/`adapters` into the parent (parent wins on collisions) |
| Display  | `src/ui/format.ts`, `LiveStatus.tsx`, `AnimatedTokens.tsx` | Pure formatting + ink components (incl. live token counter)                                                                                                                                                                                                    |
| Types    | `src/types/`                                               | Shared interfaces (`agent`, `config`, `output`, `scoring`, `report`)                                                                                                                                                                                           |

### Adapter Pattern

Built-in adapters split into two factories. NDJSON-style adapters (`claude-code`, `codex`) are created via `createAgentAdapter(spec)` from `src/adapters/base/agent-adapter.ts`. ACP-based adapters (`claude-sdk`, `codex-sdk`, `gemini`, `goose`, `opencode`, `qwen-code`, `stakpak`, `blackbox`, `fast-agent`, `mistral-vibe`, `factory-droid`, `poolside`, `vtcode`, `cursor-agent`, `auggie`, `kimi`, `openhands`, `cline`, `kiro-cli`, `kilo`, `qoder`) are created via `createAcpBasedAdapter(spec)` from `src/adapters/base/acp-adapter.ts`. Each adapter is a plain factory function (e.g. `createGeminiAdapter()`) that returns an `AgentAdapter` -no classes, no inheritance. The factory owns the shared plumbing:

- Spawn + stdin.end + cleanup registration (SIGTERM on Ctrl-C)
- 10-minute timeout → SIGTERM → SIGKILL after 5s grace (timer cleared on clean exit)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netlify/axis](https://github.com/netlify/axis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
