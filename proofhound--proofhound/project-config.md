---
trigger: always_on
description: The ProofHound open-source edition targets self-hosted scenarios, providing a single-workspace prompt lifecycle toolset: prompt versions, dataset regression testing, experiments, optimizations, canary releases, production releases, run results, annotations, and rollbacks.
---

# ProofHound

The ProofHound open-source edition targets self-hosted scenarios, providing a single-workspace prompt lifecycle toolset: prompt versions, dataset regression testing, experiments, optimizations, canary releases, production releases, run results, annotations, and rollbacks.

The repository keeps thin abstractions such as `project_id`, `ProjectContext`, `ActorContext`, and `accessControl` for the local single-project data boundary and future external control plane integration; product documentation narrates around a single workspace by default and does not expand on a control plane feature list.

This repository carries only OSS self-hosted capabilities. Future SaaS / control plane capabilities are carried by a separate repository; this repository's architecture may leave clean, thin, currently usable interface hooks, but must not pre-embed modules, features, dependencies, or product entry points that are currently useless for the sake of SaaS.

> Solo project (ZiqiXiao). Codex assists with implementation, ZiqiXiao holds all decision-making authority. When uncertain, ask first; do not rehearse all the way to the end only to discover the direction was wrong.

> `AGENTS.md` is the single source of truth. `CLAUDE.md` is a symlink to this file — edit only this file.

## 1. Tech Stack

| Layer         | Choice                                                                                                                                                                                                                                                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Frontend      | Next.js + TypeScript + Refine + shadcn/ui + Tailwind                                                                                                                                                                                                                                                                     |
| Backend       | NestJS + TypeScript monolith, split along Module boundaries                                                                                                                                                                                                                                                              |
| Database      | Native PostgreSQL + Drizzle ORM, schema prefix `ph_*`                                                                                                                                                                                                                                                                    |
| Auth          | Dual-channel HTTP entry (API `Authorization: Bearer ph_*` user token / UI deployment-layer trusted header or LOCAL_ACTOR fallback); MCP entry user token; Webhook entry per-connector webhook token; OSS ships no built-in login system, deployment forms A/B/C detailed in [08](docs/specs/08-adapter-extension-points.md) |
| Storage       | OSS stores datasets / run results inline in PostgreSQL; object storage is an override-only concern behind the dataset-upload write adapter and the dataset-sample read adapter ([08](docs/specs/08-adapter-extension-points.md) §3.13 `DatasetUploadService` / §3.14 `DatasetSampleRepository`), never in the OSS trunk                                                                                                                                                                                                   |
| Realtime      | React Query polling + NestJS SSE (business orchestration streaming)                                                                                                                                                                                                                                                      |
| Orchestration | DBOS + BullMQ + Node.js LLM Worker                                                                                                                                                                                                                                                                                       |
| Rate limit    | Redis centralized rate limiting (RPM / TPM / concurrency)                                                                                                                                                                                                                                                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proofhound/proofhound](https://github.com/proofhound/proofhound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
