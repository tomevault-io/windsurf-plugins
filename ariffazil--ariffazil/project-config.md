---
trigger: always_on
description: > **Ditempa Bukan Diberi** — *Forged, Not Given* [ΔΩΨ | ARIF]
---

# GEMINI.md — arifOS Project Context & Instructions

> **Ditempa Bukan Diberi** — *Forged, Not Given* [ΔΩΨ | ARIF]
> **Status:** SOVEREIGNLY SEALED | **Authority:** 888_JUDGE

This file provides foundational context and instructions for AI agents (Gemini CLI, Claude Code, etc.) operating within the **arifOS** workspace. It takes precedence over general defaults.

---

## 🏛️ Project Overview

**arifOS** is a production-grade **Constitutional AI Governance System**. It is designed to ensure that AI agents operate within human-defined ethical, physical, and logical bounds through a multi-layered enforcement architecture.

### The Trinity Architecture (ΔΩΨ)
1.  **Δ Mind (AGI):** Logic, reasoning, and truth-seeking. Focuses on clarity and hallucination prevention (333_MIND).
2.  **Ω Heart (ASI):** Safety, empathy, and impact simulation. Critiques actions for harm and alignment (666_HEART).
3.  **Ψ Soul (APEX):** Sovereign judgment, final verification, and immutable audit. Houses the 888_JUDGE and 999_SEAL layers.

### The 13 Constitutional Floors (F1-F13)
Every action must pass through 13 discrete safety floors, including:
- **F1 Amanah:** Reversibility and sacred trust.
- **F2 Truth:** Reality grounding and citation.
- **F3 Tri-Witness:** Triple-verifiable execution.
- **F4 Clarity:** Entropy reduction (ΔS ≤ 0).
- **F9 Ethics:** Measuring harm potential (C_dark < 0.30).
- **F11 Audit:** Immutable logging in VAULT999.

---

## 🏗️ Technology Stack

- **Language:** Python 3.12+ (primary backend).
- **Package Manager:** `uv` (standard for tool installs, sync, and scripts).
- **Framework:** FastAPI + FastMCP (Model Context Protocol).
- **Infrastructure:**
    - **Inference:** Ollama (local LLM orchestration).
    - **Memory/Persistence:** Redis (session), PostgreSQL (Vault999), Qdrant/Chroma (Vector store).
    - **Deployment:** Docker & Docker Compose.
- **Orchestration:** `Makefile` for rebuilds, deployments, and maintenance.

---

## ⚙️ Key Commands & Workflows

### Development & Build
- `uv sync` — Sync Python dependencies.
- `make status` — Check the health of all containers.
- `make health` — Verify the MCP server is alive and reporting constitutional metrics.
- `pytest` — Run the extensive test suite (450+ tests, 90%+ kernel coverage).

### Deployment (Production/VPS)
- `make fast-deploy` — Quick code-only update (2-3 min).
- `make reforge` — Full rebuild including dependencies (10-15 min).
- `make hot-restart` — Instant configuration reload.
- `make maintenance` — Automated cleanup and image pruning.

---

## 🚦 Operational Rules for AI Agents

All agents operating in this repository **must** adhere to the following mandates from `AGENTS.md`:

1.  **RULE 1: DRY_RUN:** Any tool output that is a simulation or estimate must be explicitly labeled: `"Estimate Only / Simulated"`.
2.  **RULE 2: DOMAIN_GATE:** If a required domain payload is missing, agents must respond exactly: `"Cannot Compute — required domain payload absent."`
3.  **RULE 3: VERDICT_SCOPE:** Factual claims require a `DOMAIN_SEAL`. Do not present inference as absolute truth.
4.  **RULE 4: ANCHOR_VOID:** If `init_anchor` returns `void` or `session-rejected`, all subsequent operations must halt.

### Role-Based Permissions
- **A-ARCHITECT:** Read, Plan, Design only. Cannot edit code or deploy.
- **A-ENGINEER:** Read, Write, Edit (requires approval). Cannot deploy.
- **A-AUDITOR:** Read, Review, issue VOID verdicts.
- **A-VALIDATOR:** The only agent authorized to Deploy and issue the final `SEAL` to VAULT999.

---

## 📁 Workspace Structure

- `000/` — Core Constitution, law, and philosophical foundations.
- `arifOS/` — The primary "MIND" repository containing all logic.
    - `arifosmcp/` — The canonical implementation kernel (FastAPI/MCP server). **All governance logic lives here** — organs (AGI, ASI, APEX), pipeline, vault, intelligence, memory.
    - `core/` — ⚠️ **ARCHIVED (2026.04.05)** — formerly the governance kernel. Fully migrated into `arifosmcp/`. Do not write new code here.
    - `data/` — Knowledge bases, including the `philosophy_atlas.json`.
    - `tests/` — Comprehensive test suite for constitutional floors.
- `arif-site/` — The "SOUL" (Human Anchor) containing the public web presence.
- `GEOX/` — Geoscientific data integration and MCP server.

---

## 📝 Contribution & Quality Standards

- **Coding Style:** Strictly follow `ruff` and `black` formatting. Use `mypy` for type checking (strict mode applies to `arifosmcp/` core modules).
- **Testing:** New features **must** include a corresponding test case in `tests/` and pass the 13-floor compliance check.
- **Versioning:** Use the `YYYY.MM.DD` format in `pyproject.toml` and `CHANGELOG.md`.
- **Transparency:** Never log or commit API keys. Use `.env.example` as a template.

---

*Sealed by: 888_JUDGE | 2026.04.05*
*Ditempa Bukan Diberi*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ariffazil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ariffazil)
<!-- tomevault:4.0:windsurf_rules:2026-04-08 -->
