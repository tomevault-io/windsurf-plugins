---
trigger: always_on
description: > **Canonical Entry Point for Autonomous Agents**
---

# 🤖 Siebenwind Wiki: Agent Protocols

> **Canonical Entry Point for Autonomous Agents**
> *Compatible with: OpenAI Codex, MCP-capable IDEs/CLIs, legacy Antigravity alias*

## 🎯 Mission Statement
You are operating on the **Siebenwind Wiki**, a 20-year-old collaborative world-building project. Your goal is to preserve its history while modernizing its infrastructure.
**Repository Root:** `.`

## 📜 The Golden Rules (Non-Negotiable)

1.  **Runtime Authority**: The **ONLY** executable interface is `./7w_wiki.py`. Do not create custom scripts or use `sed`/`awk` for complex logic.
2.  **Epistemic Integrity**:
    -   Never hallucinate lore. If information is missing, use tags like `[UNGEKLÄRT]`.
    -   **Epistemic precedence is `Homepage > Quellen > Wiki Pages`.**
    -   `docs/Siebenwind_Wiki/` is the technical edit tree and publishing-facing page tree, not the highest epistemic authority.
    -   Die kanonische Vollregel fuer Drift, Pages und Praezedenz steht in [SY_DRIFT_PAGES_CONTRACT.md](System/Synapse_Board/SY_DRIFT_PAGES_CONTRACT.md).
3.  **Link Hygiene**:
    -   **NO** absolute paths (`file://`).
    -   Use `[[WikiLinks]]` for knowledge base articles.
    -   Use repo-relative paths (`System/...`) for documentation.
4.  **Coordination Central**:
    -   Every new system file MUST be registered in `System/COORDINATION_HUB.md`.
    -   Use `System/Synapse_Board/` for conflict resolution.
5.  **Agent Interop**:
    -   Respect the folder structure: `.agent/` is for internal logic.
    -   Treat `.agent/catalog/catalog.v1.json` as the canonical discovery catalog.
    -   Treat `lore_manifest.json` as a generated compatibility surface, never as the source of truth.
    -   Treat `.agents/skills/` plus `.codex/config.toml` as the Codex adapter surface, not as the source of truth.
    -   Use `.agent/config/tools.json` for machine-readable tool discovery (OpenAI-compatible schema).
    -   Use `./7w_wiki.py --help-json` for dynamic CLI introspection.
6.  **Mission Report Protocol**:
    -   Every session or task MUST end with a status report via `mail done` (if working on a specific message) or `mail post` (for general updates).
    -   Reports must be concise but include: What was done, what was verified, and what is next.
7.  **Inquisitive Protocol**:
    -   If you find an anomaly unrelated to your current task, **ASK**. Do not ignore it.
    -   Use `mail post` to query the `Coordinator` or `Technician`.
8.  **Machine-Readable First**:
    -   When available, agents MUST use CLI commands with `--json` (e.g., `advisor --json`, `audit --json`) for reliable parsing.
    -   Output parsing of human-readable text is discouraged if a JSON flag exists.
9.  **No Bridge-Placeholders by Default**:
    -   Fix links to canonical targets first.
    -   Do not ship generic bridge/stub pages as final repairs.
    -   Temporary bridge exceptions require lifecycle metadata (`bridge_mode`, `bridge_target`, `bridge_ticket`, `bridge_review_until`).

<!-- BEGIN GENERATED DRIFT CONTRACT REFERENCE -->
> Generated reference block. The surrounding narrative text remains manually maintained.
> Canonical contract: [SY_DRIFT_PAGES_CONTRACT.md](System/Synapse_Board/SY_DRIFT_PAGES_CONTRACT.md)
>
> - Epistemic precedence: `Homepage > Quellen > Wiki Pages`.
> - `docs/Siebenwind_Wiki/` is the technical edit/publish tree, not the highest truth source.
> - Technical drift is validated via `./7w_wiki.py sanitize`, `./7w_wiki.py audit`, and `./7w_wiki.py pages validate --json [--strict-links]`.
> - Deterministic contract/CI checks use `./7w_wiki.py pages validate --contract --json`.
> - `--strict` hardens the MkDocs build; `--strict-links` is the hard unresolved-link gate.
> - Generated command registries are synced by `./7w_wiki.py tech --sync-docs` / `--sync-interop`; narrative rules live in the canonical contract.
<!-- END GENERATED DRIFT CONTRACT REFERENCE -->

## 🛠️ Command Registry (Executable Capabilities)

Use `./7w_wiki.py <command>` for all operations.


<!-- BEGIN GENERATED COMMAND REGISTRY -->
| Command | Purpose | Context |
| :--- | :--- | :--- |
| `search <query> [remaining...]` | Semantic RAG search (Oracle) across wiki and source corpus. | `.agent/skills/oracle/search.py` |
| `start [--run]` | Show or run the onboarding workflow. | `.agent/workflows/start.md` |
| `test` | Run interoperability and clean-state test suites. | `.agent/scripts/test_runner.py` |
| `takeover [--run]` | Show or run the takeover workflow. | `.agent/workflows/takeover.md` |
| `handover [--run]` | Show or run the handover workflow. | `.agent/workflows/handover.md` |
| `historian [query]` | Deep lore analysis workflow or direct topic run. | `.agent/workflows/historian.md` |
| `repair` | Interactive or automatic repair of audit findings, including Pages / Roamlinks fixes. | `.agent/scripts/repair.py` |
| `audit` | Run consistency audit for duplicates, broken links, and orphaned content. | `.agent/scripts/register_check.py` |
| `index` | Manage the semantic search index. | `.agent/skills/oracle/build_index.py` |
| `index-pages` | Generate category index pages for the wiki. | `.agent/scripts/generate_wiki_indices.py` |
| `pages <status|build|validate>` | Build or validate GitHub Pages documentation and site-integrity health. | `.agent/scripts/pages_tool.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Siebenwind) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
