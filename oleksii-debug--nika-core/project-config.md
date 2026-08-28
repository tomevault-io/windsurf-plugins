---
trigger: always_on
description: This repository is the canonical source of truth. Chat history is not.
---

# Nika Core autonomous development instructions

This repository is the canonical source of truth. Chat history is not.

Before every coding cycle read: `docs/MASTER_SPEC.md`, `docs/FULL_PRODUCT_VISION_2026-08-19.md`, `docs/AUTONOMOUS_PRODUCT_FACTORY.md`, `docs/AUTONOMOUS_PRODUCT_FACTORY_ACCEPTANCE.md`, `docs/AUTONOMOUS_BUSINESS_FACTORY.md`, `docs/ROADMAP.md`, `docs/THIRD_PARTY_ADOPTION.md`, `docs/REUSE_CATALOG_2026-08-18.md`, `docs/INTELLIGENCE_REUSE_2026-08-19.md`, `docs/WORKSPACE_REUSE_CATALOG_2026-08-18.md`, `docs/WORKSPACE_REUSE_CATALOG_2026-08-19.md`, `docs/UI_ARCHITECTURE.md`, `docs/LARGE_BATCH_POLICY.md`, `docs/PARALLEL_DEVELOPMENT_POLICY.md`, `docs/AUTONOMOUS_DEVELOPMENT_PROTOCOL.md`, `docs/ACCEPTANCE_GATES.md`, `state/PROJECT_STATUS.md`, `state/PARALLEL_EXECUTION_BOARD.md`, LIVE DASHBOARD, open PRs and current CI.

Primary rule: **REUSE BEFORE REWRITE**. Search maintained upstream libraries and current official documentation before implementing a subsystem. Default decision order is **REUSE -> ADAPT -> CUSTOM (thin)**. A CUSTOM decision is invalid unless it records why maintained upstream options do not satisfy the requirement. Do not copy random or wholesale third-party source into this repository when a package dependency/adapter is sufficient. Do not add broad unused dependencies merely “for later”; graduate a candidate through a focused proof, exact license/version check and tests.

Architecture: Windows-first modular monolith with ports/adapters and versioned contracts. Nika owns task/audit/permission/product contracts; provider-neutral Model Gateway; workspace/plugin boundaries; ProductProject and ProductRepositoryGraph contracts; Deterministic Brain contracts; deterministic state/validation/dedup/safety; and the Product Journey/Product Factory gates. Language models are replaceable capabilities, not the platform kernel. Agent orchestration sits behind `AgentRuntimePort`; third-party framework/model/planner/coding-worker types must not leak into Nika domain APIs.

Intelligence truth: Nika has four distinct paths that must not be conflated: (1) model-free Deterministic Brain, (2) embedded local model with Microsoft Foundry Local as primary Windows adapter and measured alternatives such as llama.cpp/ONNX Runtime GenAI, (3) external local model servers such as Ollama, and (4) allowed cloud/API providers. Installing or testing one does not award evidence for the others.

Runtime truth: M2 selected and integrated LangGraph behind `AgentRuntimePort`. Microsoft Agent Framework remains a secondary migration/interop candidate. Do not run multiple competing orchestration kernels in production unless a new measured proof demonstrates a concrete requirement.

Full-product truth: historical Core percentages do not equal completion of the expanded Full Product Vision. A backend subsystem is not finished until its actual packaged Windows user journey is connected and proven. Telegram is removed from active roadmap scope; old Telegram references in historical reuse documents are non-binding unless a future explicit user request reintroduces such a workspace/ProductProject.

Autonomous Product Factory truth: **build the factory, not every possible product**. Do not interpret examples such as a messenger, social network, screen-reader-like product, browser-agent platform or business application as requirements to hard-code those products into Core. The requirement is a durable ProductProject lifecycle that can research, specify, compose a specialist team, create/connect one or more repositories, implement in isolation, independently review/test, package, deploy under policy and maintain the resulting product. A large ProductProject may run for days/weeks/months and must not depend on chat memory.

Product-vs-Toolsmith truth: Toolsmith closes a narrow missing capability for an existing task. Product Factory owns complete product/system goals. Do not force a large product request into one oversized CodingJob or claim Product Factory completion from a coding-worker demo.

Dynamic-team truth: team roles are derived from ProductProject scope, dependencies and risk. One worker may cover several roles for a small project; a large project may need independent research, product, architecture, security, backend, frontend, Windows/mobile, QA, accessibility, DevOps/release and support roles. Agent count is not a success metric. New specialization may be added during execution without widening the ProductProject permission ceiling.

Research-to-Product truth: Universal Research is the canonical evidence/research layer for Product Factory and Business Factory. Where the user requests discovery/market/competitor analysis, research evidence and user/policy decisions must become versioned ProductProject inputs without manual copy/paste. Do not create a second research engine for Product Factory.

Repository/deployment truth: Product Factory may own one repository or a multi-repository ProductRepositoryGraph. Source generation is not product completion. Build/package, approved staging/deployment, health proof, rollback and maintenance are part of the product lifecycle where applicable. Use provider-neutral adapters and execution nodes; do not assume the local Windows laptop can build every platform target.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Oleksii-debug/Nika-Core](https://github.com/Oleksii-debug/Nika-Core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
