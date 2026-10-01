---
trigger: always_on
description: Documentation is the core constraint of this project. Treat `/AGENTS.md` (this file) as the **Master Runtime Contract** and bootloader for AI agents and human developers working on the open-source components of the SanadAgent system.
---

# Sanad Agent Repository Contract

Documentation is the core constraint of this project. Treat `/AGENTS.md` (this file) as the **Master Runtime Contract** and bootloader for AI agents and human developers working on the open-source components of the SanadAgent system.

> [!IMPORTANT]
>
> ### 🧭 Documentation is the most important part of this project
>
> Treat every `AGENTS.md` file as a **Runtime Contract**, not as optional notes. Poor documentation causes AI agent behavior drift, architecture drift, and incorrect changes in the wrong layers.
>
> 1. **Functional Discoverability:** For a complete, curated index of all codebase features, components, and guides, read **[docs/llms.txt](docs/llms.txt)** immediately.
> 2. **Operational Rules:** Plan, coordinate, and execute tasks using Git Worktrees, dynamic port offsets, and multi-agent systems.

---

## 1. Documentation Hierarchy

This repository uses a nested web of contracts. The closer the document is to the code, the more technical it should be:

* **`AGENTS.md`** (this file / repository root) — Main Master Contract governing all open-source components.
* **[client/AGENTS.md](client/AGENTS.md)** — Owns Flutter client UI contracts, widgets, and state registries (Flutter).
* **[agent/AGENTS.md](agent/AGENTS.md)** — Owns local Dart daemon interfaces, MCP servers, and background services (Dart).

### Rules for AI Agents and Developers

* **Always read the local `AGENTS.md`** file governing a subdirectory before making any edits in that directory.
* **Always update the relevant docs in the same session as the code change.** This is a strict Definition of Done (DoD).
* **Review the closest owning `AGENTS.md`** for every changed area. Update it only when the change alters a durable law, ownership boundary, invariant, or makes an existing statement stale; do not edit contracts merely to record implementation activity.
* **Keep higher-level docs abstract** where appropriate and push implementation detail down into local leaf docs.
* **Keep lower-level docs concrete, explicit, and practical.**
* **Remove stale or contradictory documentation immediately** to prevent behavior drift.
* **Use `README.md` only as the public product source of truth** for quick starts and pitches. **Do not let `README.md` become a competing implementation contract;** durable architecture and development contracts belong here in `AGENTS.md` and their nested child contracts.
* **Use `fvm` for all Flutter/Dart operations.** Never execute global flutter/dart commands.
* **All referenced file paths must be relative to the workspace root.**

---

## 1.1. The Strict Separation Pact

To prevent instruction drift and save session context, we enforce a strict separation between three knowledge streams in the repository:

1. **`AGENTS.md` files (The LAWS):** Core operational policies and rigid coding guidelines (e.g., "always use relative paths", "use fvm"). They MUST NOT contain design explanations or execution commands.
2. **Agent Skills in `.agents/skills/` (The TOOLS & SOPs):** Step-by-step developer/agent procedures and execution commands (e.g., worktree and testing workflows). They MUST NOT contain system design or database schemas.
3. **The `docs/` Directory (The DESIGN & GUIDANCE):** Technical design, product/UX specifications, APIs, schemas, operations, QA runbooks, and user/operator guidance. Docs MUST NOT establish coding/runtime laws for agents or duplicate developer-agent execution SOPs from skills. Commands are allowed only when necessary for user/operator, deployment, troubleshooting, or runbook content—not as instructions governing agent development behavior.

## 1.2. The Living Project Wiki & Incremental Documentation

The project documentation lives in the `docs/` directory under the 5-Pillar Architecture:

1. **`docs/product/`:** PRD, UX scenarios, wireframes.
2. **`docs/technical/`:** Database schemas, API specs, communication protocols.
3. **`docs/agent_engine/`:** Prompts, agent tools, function calling contracts.
4. **`docs/operations/`:** Deployment, n8n workflows, Docker configurations.
5. **`docs/qa_maintenance/`:** Test scripts, QA scenarios, troubleshooting guides (Runbooks).

* **Incremental Reverse Documentation:** Any task modifying or adding code to a file must include writing/updating the corresponding documentation page in `docs/` as part of its Definition of Done (DoD).

---

## 2. Directory Structure

The `sanad-agent` repository is structured as follows:

* **`agent/`**: Dart background daemon.
* **`client/`**: Flutter UI desktop client.
* **`docs/`**: Feature plans, product specifications, and markdown documentation.
* **`release/`**: The versioned release contract, generated-output policy, and shared Dart contract package under `release/contract/` for manifest, checksum, and Appcast models consumed by the agent, client, and release tooling.
* **`shared/`**: Focused pure-Dart runtime primitives consumed by both the agent and client; each package owns a local contract and must remain independent of presentation and agent execution domains.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EastStarAI/sanad-agent](https://github.com/EastStarAI/sanad-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
