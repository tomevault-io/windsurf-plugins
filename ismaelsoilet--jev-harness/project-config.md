---
trigger: always_on
description: **[ 🇬🇧 English ](AGENTS.md) | [ 🇧🇷 Português ](AGENTS.pt-BR.md)**
---

# 🤖 AGENTS.md: AI Agent Constitution and Operational Manual

**[ 🇬🇧 English ](AGENTS.md) | [ 🇧🇷 Português ](AGENTS.pt-BR.md)**

> **ATTENTION:** ANY autonomous artificial intelligence agent (Claude Code, OpenAI Codex / Astra-Codex, Pi, Oh My Pi, CommandCode, Cursor, Antigravity IDE, OpenCode, Windsurf, Zed, Devin, Aider) opening this repository **MUST READ THIS DOCUMENT** before planning, modifying code, or executing changes.
> 
> *Project Version: v0.2.0 — Synchronized across all 4 official registries.*

---

## 🧭 Quick Index of Modular Conventions and Rules

This document serves as the primary entry portal. The repository organizes its specialized and self-contained rules under [`.agents/rules/`](.agents/rules/):

1. 🏛️ **[System Architecture Blueprint](.agents/rules/01_project_blueprint.md)**: Directory mapping, tri-runtime layout (Python, TS, Rust), data pipelines, and zero-dependency contracts.
2. 🧠 **[Core Software Engineering Principles](.agents/rules/02_software_engineering_principles.md)**: Karpathy principles, Fable loop, Kahneman System 1 vs 2, Unix philosophy, and anti-Frankenstein architecture.
3. 🌐 **[Model Governance & 2026 Frontier Registry](.agents/rules/03_model_governance_and_frontier_registry.md)**: Golden rule against obsolete models, mandatory daily web research, provider dialects, and direct model safeguards.
4. 🛡️ **[Exhaustive Testing & Absolute Truthfulness](.agents/rules/04_testing_and_truthfulness.md)**: Zero-trust posture, prohibition of tautological tests, 626-test battery.
5. 🚀 **[Release Protocol & Quad-Sync Synchronization](.agents/rules/05_release_and_quad_sync_protocol.md)**: Synchronous pipeline across 4 registries (GitHub, PyPI, npm, Crates.io).
6. 📝 **[Code Style and Language Conventions](.agents/rules/06_code_style_and_conventions.md)**: Strict standards for Python (pure stdlib), TypeScript (native ESM), and Rust (Tokio 2021).
7. 🔌 **[MCP Quality & TDQS Standards](.agents/rules/07_mcp_quality_and_tdqs_standards.md)**: Glama TDQS A+ (5.0) requirements, verb_noun canonical naming, MCP annotations, usage guidelines, and tri-runtime parity.
8. 🤖 **[Universal Agent Implementation Guide](docs/AGENT_INTEGRATION_GUIDE.md)** ([Português](docs/AGENT_INTEGRATION_GUIDE.pt-BR.md)): Step-by-step playbook to plug Jev Harness via MCP, CLI, or native SDK into any project in 2 minutes.
9. 🗺️ **[System 1.5 — Architecture, Opportunities and Implementation Plan](docs/system_1_5/SYSTEM_1_5_IMPLEMENTATION.md)**: positioning between System 1 (Jev) and System 2, verified facts, ecosystem comparison and the phased implementation plan. **Not a release plan** — the next release (v0.2.0) follows the protocol in `.agents/rules/05`.
10. 📚 **[Documentation Map & Catalog](docs/README.md)** ([Português](docs/README.pt-BR.md)): Complete documentation inventory and navigation guide.

---

## 🏛️ 1. Project Blueprint (System Blueprint)

`jev-harness` solves the most expensive problem in agentic computing: **wasting frontier tokens on trivial mechanical failures and circular doom loops**.

### Tri-Runtime Architecture with Strict Semantic Parity:
* **Python Core (`src/jev_harness/`)**:
  - `client.py`: Ultra-resilient HTTP client using pure standard library (`urllib.request`), dynamic timeout, and offline heuristic simulation in **< 500µs**.
  - `gates.py`: Implementation of the 6 semantic decision gates (`triage_test_failure`, `should_abort_trajectory`, `route_model_tier`, `verify_step_completion`, `modulate_reasoning_effort`, `should_nudge_continuation`).
  - `mcp_server.py`: Universal stdio MCP server for direct integration with Cursor, Claude Desktop, and Antigravity IDE.
  - `cli.py`: Command-line interface (`jev-harness`) in strict compliance with Unix pipes.
  - `session.py`: Session telemetry, ROI calculation, and atomic concurrency lock (`fcntl.flock`).
* **Rust Crate (`packages/rust/`)**:
  - High-performance implementation in stable Rust (Tokio + Serde), providing the `jev_harness` crate, native stdio MCP server (`mcp.rs`), and standalone CLI binaries `jev` and `jev-harness`.
* **TypeScript Package (`packages/ts/`)**:
  - Native npm package `@ismaelsoilet/jev-harness` supporting Node.js, Bun, and Deno, exporting a typed SDK, stdio MCP server (`mcp.ts`), and CLI executable via `npx`.

---

## 🧠 2. Core Software Engineering Principles

Every agent operating in this repository must guide its decisions by five non-negotiable pillars:

### 1. Karpathy Principles for LLM Coding
* **Think Before Coding**: Do not assume. Do not conceal confusion. State assumptions and trade-offs explicitly before applying edits.
* **Simplicity First**: Deliver the minimum code that solves the current problem with excellence. Zero speculative code. No complex design patterns (Factory, Strategy) for a single use-case.
* **Surgical Changes**: Touch strictly what was requested. Never refactor, reformat, or change quotes in adjacent code outside the requested scope.
* **Goal-Driven Execution**: Turn tasks into verifiable success criteria. When fixing a bug, reproduce it with an automated test before implementing the patch.

### 2. Absolute Truthfulness and Zero-Trust Posture
* **NEVER TRUST BLINDLY, ALWAYS TEST AND VERIFY.**
* **IF SOMETHING SEEMS UNUSUAL, INVESTIGATE DOWN TO THE ROOT CAUSE.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ismaelsoilet/jev-harness](https://github.com/ismaelsoilet/jev-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
