---
trigger: always_on
description: This file is the standing project context for Codex and other coding agents working in this repository. Read it before changing code. Also read the relevant source and tests for the area being changed. The repository is the source of truth for current behavior; documents under `docs/` describe product and architecture intent and may include future work.
---

# ReleaseGuard AI Project Context

This file is the standing project context for Codex and other coding agents working in this repository. Read it before changing code. Also read the relevant source and tests for the area being changed. The repository is the source of truth for current behavior; documents under `docs/` describe product and architecture intent and may include future work.

## Product Goal

ReleaseGuard AI is an AI Release Intelligence Agent for product managers, product owners, and release owners. It helps teams determine whether a release caused a product problem, identify affected metrics and user segments, connect symptoms to release changes and user feedback, test competing hypotheses with evidence, recommend a controlled response, and place external write actions behind explicit human approval.

The product focuses on product release risk: product metrics, user feedback, release context, limited technical signals, and PM decisions. It is not a general-purpose SRE agent, infrastructure debugger, or autonomous remediation system.

## Version And Provenance

- Current migrated product version: **v17**.
- Project identity: **ReleaseGuard AI / risk-command-center**.
- Source branch at packaging time: `main`.
- Original deployed v17 HEAD recorded by the migration package: `75cbe4140aa0ff5e079df17fead96167836129db`.
- The migration archive intentionally excluded the original `.git` database.
- Any commit created after importing this archive is a new local migration commit. Do not claim that it is identical to the original v17 HEAD.
- The v17 archive was produced from the deployed Git tree and excludes dependencies, build outputs, runtime state, TypeScript caches, local databases, and real environment values.

## Completed Capabilities

### Phase 1: Investigation Runtime

- Persistent, auditable `InvestigationRun` lifecycle.
- Structured `ToolCall`, `ToolResult`, `Evidence`, `Diagnosis`, and `ProposedAction` records.
- Server-owned run state and recovery after page refresh.
- Deterministic Android 7.3.0 investigation fixture.
- OpenAI-compatible live model configuration, including DeepSeek and Kimi-compatible endpoints.

### Phase 1.5: Approval And Action Hardening

- Persistent approval tied to one run and one specific proposed action.
- Server-side approval validation and frozen action arguments.
- GitHub Issue creation represented and audited as a runtime tool call.
- Replay/deduplication protection and audit events.
- Explicit reject and `CLOSED_NO_ACTION` behavior.
- Successful approved action transitions the run to `WAITING_VERIFICATION`.

### Phase 2: Risk Detection And Analytics

- Metric buckets and deterministic Android 7.3.0 analytics fixture.
- Deterministic anomaly detection using dynamic baselines, minimum sample sizes, and consecutive-window rules.
- Persistent `RiskEvent` linked to a `Release` and an `InvestigationRun`.
- Formal `get_release`, `query_metric`, and `segment_metric` tool contracts.
- Investigation launch from a detected `RiskEvent`.

### Phase 3: Agent Loop, Chat, And Hybrid RAG

- A shared `InvestigationPlanner` contract implemented by `LLMInvestigationPlanner` and `DeterministicInvestigationPlanner`.
- One service-side `AgentLoop` for both planner types.
- Persistent iterations, public trace events, hypotheses, evidence links, confidence calculation, messages, diagnosis revisions, and approval snapshots.
- Continue Investigation flow from `WAITING_APPROVAL`, including withdrawal of the pending snapshot and return to `RUNNING`.
- Human-Agent chat integrated with the same investigation runtime.
- Feedback and historical incident retrieval with chunking, metadata, embeddings, lexical/vector search, and hybrid fusion.
- Workers AI and Vectorize production path, plus an explicit deterministic local/CI fallback.
- Deterministic Agent Runtime regression coverage and RAG evaluation.
- Production-bundle guard for deterministic planner construction.

## Core Architecture

- `app/`: product UI and server API routes for investigations, messages, approvals, GitHub connection, and GitHub Issue actions.
- `lib/investigation/`: investigation state, runtime, shared planner contract, LLM and deterministic planners, AgentLoop, tools, confidence, chat, revisions, approval/action runtime, repositories, and stores.
- `lib/analytics/`: releases, metric data, RiskEvent persistence, and analytics access.
- `lib/risk-detection/`: deterministic baseline and anomaly detection policy.
- `lib/retrieval/`: feedback and incident retrieval contracts, tokenization, embeddings, lexical/vector/hybrid retrieval, and fallback runtime.
- `lib/fixtures/`: deterministic Android 7.3.0 fixture data.
- `db/`: D1/Drizzle schema and database adapters.
- `drizzle/`: ordered SQL migrations. Preserve migration order and never rewrite an applied migration casually.
- `worker/`: Cloudflare Worker entry point and runtime binding integration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eason4real/releaseguard-ai](https://github.com/Eason4real/releaseguard-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
