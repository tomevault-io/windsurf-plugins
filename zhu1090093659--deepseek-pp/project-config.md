---
trigger: always_on
description: These instructions apply to the entire repository. `AGENTS.md` is the sole project-level agent instruction truth source for this project.
---

# DeepSeek++ Project Agent Instructions

## Scope

These instructions apply to the entire repository. `AGENTS.md` is the sole project-level agent instruction truth source for this project.

## Product Context

DeepSeek++ is a WXT/React/TypeScript MV3 browser extension that adds agentic memory, Skills, tools, automation, and related workflows to DeepSeek. Its core runtime model is:

- intercept DeepSeek Web requests and streams from the page runtime;
- augment prompts with memory, Skill, preset, project, and tool context;
- parse tool-call output and execute approved capabilities through extension-owned boundaries;
- persist user state in IndexedDB and browser storage;
- integrate optional MCP, Native Host, sync, sandbox, Side Panel, and floating-chat surfaces.

Chrome, Edge, and Firefox on desktop are the only supported product targets. Android, mobile WebView shells, and mobile packages are outside the current product scope.

## Truth Sources

- `AGENTS.md` — stable project rules, architecture invariants, and recurring engineering constraints.
- `docs/archives/deepseek-pp-reliability-compatibility-refactor/progress/MASTER.md` — completed core-refactor record, GitHub Issue/Milestone mapping, validation evidence, and final known gap.
- `docs/archives/deepseek-pp-reliability-compatibility-refactor/analysis/project-overview.md` — confirmed PC-only scope and compatibility boundary established by the completed refactor.
- `docs/archives/deepseek-pp-reliability-compatibility-refactor/analysis/module-inventory.md` and `docs/archives/deepseek-pp-reliability-compatibility-refactor/analysis/risk-assessment.md` — final architecture evidence and risk basis for that run.
- `docs/compatibility/README.md` and its linked registries — stable prompt, runtime, persistence, browser, integration, and historical-data contracts established by the completed refactor.
- `docs/archives/deepseek-pp-reliability-compatibility-refactor/plan/task-breakdown.md` and `docs/archives/deepseek-pp-reliability-compatibility-refactor/plan/dependency-graph.md` — archived task ownership, dependencies, and execution lanes.
- `package.json`, `wxt.config.ts`, and GitHub workflows — executable build, test, manifest, and release contracts.
- `docs/releases/<version>.md` — exact public release/update notes for that version.

When prose and executable behavior disagree, verify the code and tests, then update the stale document in the same task. Do not create a second source of truth to bridge the mismatch.

## Collaboration Rules

- Once the core scope is aligned, make routine technical decisions and proceed. Ask only when a new choice materially changes behavior, compatibility, risk, or authorization.
- Prefer root-cause structural fixes over symptom patches. Remove duplicate logic, obsolete gates, dead branches, and hidden fallbacks instead of layering another path beside them.
- Keep changes scoped to the active Issue. Preserve unrelated and pre-existing working-tree changes.

## Architecture and Compatibility Invariants

- Preserve prompt byte output, tool XML tags, inline-agent continuation/finalization semantics, and user-visible behavior unless an Issue explicitly authorizes a contract change.
- Preserve storage keys, IndexedDB names/tables/identity, supported schema versions, sync/export records, runtime message names, MAIN/content bridge records, MCP contracts, Native Host contracts, and Chrome/Edge/Firefox degradation semantics.
- Every schema change requires an explicit, deterministic, idempotent migration. Unknown future versions and corrupt data must fail visibly without overwriting the original state.
- Maintain one authoritative router, validator, policy, and persistence truth for each concept. Delete the superseded path as soon as its consumer migrates.
- Define contracts before implementations. Contract modules must not import concrete browser, DOM, provider, or entrypoint implementations.
- Introduce only narrow environment ports. A new port must gain a production consumer in the same task; otherwise remove it. Do not expand the existing broad platform abstraction without a real consumer.
- Cross-runtime and external-I/O contracts must be serializable and validated at the receiving trust boundary.
- Before privileged runtime, Port, MessagePort, or frame dispatch, derive authority only from browser-provided sender, tab, frame, document, WindowProxy, and receiver-owned correlation state, then run the direction-specific codec. Message-declared source, tab, frame, session, or request IDs are routing claims, not identity; MAIN-world payloads remain untrusted. Opaque sandbox `postMessage('*')` is allowed only with exact source/origin checks and strict request correlation.
- Every production tool execution must pass through the runtime authorization path before payload rehydration or provider execution. Page/model calls require a background-owned grant that binds the receiver-owned document/session, advertised descriptor security snapshot, canonical provider/mode/risk, request identity, and one-time call reservation; caller-supplied ToolCall metadata is never authorization evidence.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhu1090093659/deepseek-pp](https://github.com/zhu1090093659/deepseek-pp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
