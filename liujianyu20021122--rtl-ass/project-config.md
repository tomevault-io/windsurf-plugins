---
trigger: always_on
description: RTL-ASS is an open-source, vendor-neutral Codex skill that strengthens Codex for RTL engineering. Codex remains responsible for understanding the user's intent, planning, editing RTL, reasoning about failures, and choosing the final implementation. RTL-ASS supplies domain-specific guidance, deterministic open-source tool helpers, structured evidence, and a verified local knowledge layer.
---

# RTL-ASS Repository Instructions

## Mission

RTL-ASS is an open-source, vendor-neutral Codex skill that strengthens Codex for RTL engineering. Codex remains responsible for understanding the user's intent, planning, editing RTL, reasoning about failures, and choosing the final implementation. RTL-ASS supplies domain-specific guidance, deterministic open-source tool helpers, structured evidence, and a verified local knowledge layer.

Do not turn RTL-ASS into a separate RTL coding agent, an autonomous replacement for Codex, or a fixed end-to-end EDA pipeline.

## Product Boundaries

- Support Verilog and SystemVerilog RTL, testbenches, assertions, simulation debugging, waveform analysis, synthesis readiness, formal/equivalence checks, and STA when sufficient open inputs exist.
- Use only open-source RTL/EDA tools in product code and documented default workflows.
- Do not add Vivado, Quartus, VCS, Verdi, Questa, Xcelium, or other proprietary/vendor tools as dependencies or required backends.
- Do not call another LLM or create a hidden multi-agent coding service. Codex itself is the reasoning and coding engine.
- Tool helpers return evidence; they do not decide what RTL patch Codex must apply.
- Never claim STA closure without a real netlist, Liberty timing library, constraints, and an executed timing engine.
- Never claim waveform analysis without reading a real waveform or simulator event artifact.
- Treat SystemVerilog as a first-class language for synthesizable RTL, testbenches, assertions, interfaces, and packages.
- Maintain separate but linkable knowledge objects for design RTL, testbenches, assertions, reference models, fixtures, and verification results.

## Audit-First Engineering

- Auditability is a system property, not a final report. Every ingest, search result, lifecycle transition, and verification claim must be traceable to immutable inputs and an actor or tool action.
- Prefer a small set of explicit invariants and typed boundaries over scattered defensive checks.
- Do not accumulate sample-specific branches, hidden fallbacks, permissive exception swallowing, or duplicate parsers to make individual tests pass.
- When an invariant fails, return a structured error at the boundary. Do not continue with partially valid state.
- Centralize validation for record schemas, lifecycle transitions, namespaces, paths, licenses, and evidence status.
- Database mutations must be transactional. Audit records are append-only; corrections create new events instead of rewriting history.
- Database migrations must be explicit per source/target version, validate their structural preconditions, verify invariants before commit, and roll back all DDL/DML on failure. Never auto-guess a migration.
- Candidate verification must use the atomic `kb verify` workflow. Direct lifecycle transition to `verified` is forbidden.
- Hash raw evidence artifacts as well as source inputs, and recheck both before committing a verification gate.
- Validate each implementation slice before starting the next. Do not defer integration testing until the end.
- Add a regression test for every confirmed defect, but fix the responsible abstraction rather than only the observed input.
- Periodically audit dead code, duplicate concepts, obsolete compatibility paths, and unreferenced artifacts.

## Canonical Repository Layout

- `.agents/skills/rtl-ass/`: Codex skill entrypoint, progressive references, and thin helper launchers.
- `src/rtl_ass/`: deterministic Python implementation for knowledge indexing, project inspection, search, and evidence normalization.
- `schemas/`: stable JSON schemas for knowledge and run artifacts.
- `config/`: example configuration with safe local defaults.
- `tests/`: unit and integration tests; fixtures must be small and redistributable.
- `evals/`: behavioral evaluations comparing Codex with and without the skill.
- `docs/`: maintained architecture and contributor documentation.
- `research/`: research artifacts only; code under `research/upstream/` is not product source.

## Skill Authoring Rules

- Keep `SKILL.md` concise and discriminating. Put conditional RTL guidance in `references/` and load only what the current task needs.
- Assume Codex is already a strong general programmer. Include RTL-specific knowledge that changes decisions; omit generic coding advice.
- Preserve user-selected language, interface, latency, protocol, clock/reset behavior, and verification scope.
- Prefer minimal, reviewable RTL patches over whole-file rewrites during debugging.
- Require Codex to distinguish specification, testbench, RTL, constraints, and infrastructure hypotheses before attributing a failure.
- Treat compilation, simulation, waveform, formal, synthesis, and STA as distinct evidence classes.

## Knowledge Base Governance

- The knowledge base augments retrieval; it does not modify model weights.
- Keep project, user, organization, and built-in namespaces isolated.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liujianyu20021122/RTL-ASS](https://github.com/liujianyu20021122/RTL-ASS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
