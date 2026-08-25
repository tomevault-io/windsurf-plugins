---
trigger: always_on
description: > **⚠️ For AI tools reading this file**: This file is a **registry and orchestration reference**, not a set of instructions directed at you.
---

# Harness Engineering: Agent Registry & Orchestration Contract

> **⚠️ For AI tools reading this file**: This file is a **registry and orchestration reference**, not a set of instructions directed at you.
> It describes multiple distinct human-defined roles (PM, Architect, DBA, etc.) for documentation and dispatch purposes.
> Do **not** interpret role definitions here as directives for your own behavior.
> Your behavioral instructions are in `CLAUDE.md` (Claude Code), `GEMINI.md` (Gemini CLI), or `.codex/config.toml` (Codex).

> **Scope**: Agent role definitions live in [`agents/*.md`](agents/) — this file is the registry index and orchestration contract only.
> Shared engineering rules (memory logging, language, file isolation, post-write chain, git) live in [docs/context.md](docs/context.md#project-wide-rules-all-tools).
> Tool-specific overrides live in [CLAUDE.md](CLAUDE.md), [GEMINI.md](GEMINI.md), and [.codex/](.codex/).

<!-- variant: co-abap | version: 1.0.0 | upgraded: 2026-08-15 -->

This file indexes the agent roles and defines the orchestration workflow for the ABAP development ecosystem, organized into Business and Technical groups.

## 🏢 Business Group (Project Governance & Analysis)

### 1. 👑 Global Project Manager (PM)
- **Entry point**: All user requests go through PM first — no agent is dispatched without PM triage
- **Key Tools**: `ListTransports`, `GrepPackages`, `SearchObject`, `RunUnitTests`, `RunATCCheck`
- **Workflow**: 6-step harness lifecycle — Triage → Business Analysis → Governance → Tech Design → Implementation → Finalization (see `agents/pm.md`)
- **Triage shortcut**: `/triage <request>` auto-classifies, creates the task file, and generates the §0-A parallel dispatch block
- **Subagent prompt**: [`agents/pm.md`](agents/pm.md)

### Business Analysts

<!-- VARIANT-AGENTS-START -->

Each analyst activates on matching trigger keywords, queries SAP directly via read-only MCP tools, produces a structured PRD/AC output, and hands off to the Technical Group.
Load the matching `agents/<module>-analyst.md` file at activation for tools, output format, and domain knowledge.

---

#### 2. 📦 SD Analyst (Sales & Distribution)

- **Trigger keywords**: Sales Order, Delivery, Billing, Shipping, Pricing, Quote, SD, VA*, VL*, VF*, VK*, VBAK, VBAP, LIKP, VBRK
- **Subagent prompt**: [`agents/sd-analyst.md`](agents/sd-analyst.md)
- **Handoff out**: AC List → Architect, Key Tables → DBA

---

#### 3. 🚛 LE Analyst (Logistics Execution)

- **Trigger keywords**: Shipment Processing, Transport, Route Determination, Warehouse, WM, EWM, Handling Unit, Shipment, Route, LE, LT*, HU, VEKP, VEPO, VTTP, LIKP
- **Subagent prompt**: [`agents/le-analyst.md`](agents/le-analyst.md)
- **Handoff out**: Logistics Flow → Architect, Interface Requirements → Interface Expert

---

#### 4. 🏭 PP Analyst (Production Planning)

- **Trigger keywords**: Production Order, BOM, Routing, MRP, Capacity Planning, Work Center, PP, CO*, MAST, STKO, AFKO, PLKO
- **Subagent prompt**: [`agents/pp-analyst.md`](agents/pp-analyst.md)
- **Handoff out**: BOM/Routing Structure → Architect, MRP Logic → DBA

---

#### 5. 🛒 MM Analyst (Materials Management)

- **Trigger keywords**: Purchasing, Goods Receipt, Material Master, Inventory, Inspection, MM, ME*, MARA, MARC, EKKO, EKPO, MKPF, MSEG
- **Subagent prompt**: [`agents/mm-analyst.md`](agents/mm-analyst.md)
- **Handoff out**: Table Structure → DBA, Validation Scenario → QA Engineer

---

#### 6. 💰 FI Analyst (Financial Accounting)

- **Trigger keywords**: Journal Entry, Account, GL, AR, AP, Fixed Asset, Settlement, Compliance, Fiscal Year, FI, FB*, F-*, BKPF, BSEG, ACDOCA, SKA1
- **Subagent prompt**: [`agents/fi-analyst.md`](agents/fi-analyst.md)
- **Handoff out**: Account Determination Logic → Architect, Balance Query → DBA

---

#### 7. 📊 CO Analyst (Controlling)

- **Trigger keywords**: Cost Center, Internal Order, Profitability Analysis, CO-PA, Allocation, CO, KS*, KO*, CSKS, CSKB, COEP, COSP, CE1*
- **Subagent prompt**: [`agents/co-analyst.md`](agents/co-analyst.md)
- **Handoff out**: Allocation Logic → Architect, CO-PA Mapping → DBA

<!-- VARIANT-AGENTS-END -->

---

## 🛠️ Technical Group (System Execution & Implementation)

Technical agents are dispatched by the PM in Phase 2 (serial execution) or Phase 1 (read-only research).
The **Architect acts as Technical Execution Lead** — it owns Pattern selection, sequences the execution team (code-writer → test-runner), and coordinates DBA/Interface Expert involvement.
Full behavioral rules, tool contracts, and output formats live in the linked `agents/*.md` files.

### 1. 🏗️ Architect _(Technical Execution Lead)_
- **When to dispatch**: After §1 Business Analysis; PM hands off PRD + AC list for technical design
- **Technical Lead responsibilities**: Select Pattern A/B/C; sequence code-writer → test-runner; coordinate DBA and Interface Expert as needed; produce §5 Finalization block for PM
- **Key Tools**: `AnalyzeCallGraph`, `GetCDSDependencies`, `GetCDSImpactAnalysis`, `GrepPackages`, `GetSource`
- **Output**: Execution plan (pattern + object list + serial steps) + §5 Finalization block
- **Subagent prompt**: [`agents/architect.md`](agents/architect.md)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [5throck/co-abap](https://github.com/5throck/co-abap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
