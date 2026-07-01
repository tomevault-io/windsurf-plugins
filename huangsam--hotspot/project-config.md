---
trigger: always_on
description: This document provides high-level architectural context and domain concepts for the Hotspot CLI tool to assist AI agents. For detailed implementation, struct definitions, and execution flows, agents should directly read the source code in `cmd/`, `core/`, and `schema/`, as Hotspot uses standard Go CLI patterns (Cobra/Viper) that are easily discoverable.
---

# Hotspot CLI Agent Documentation

This document provides high-level architectural context and domain concepts for the Hotspot CLI tool to assist AI agents. For detailed implementation, struct definitions, and execution flows, agents should directly read the source code in `cmd/`, `core/`, and `schema/`, as Hotspot uses standard Go CLI patterns (Cobra/Viper) that are easily discoverable.

## Architecture & Data Flow

Hotspot is a Git repository analysis tool that identifies code hotspots through various scoring algorithms. Whether invoked via the traditional CLI or the MCP server, it follows a unified analysis pipeline:

**Analysis Pipeline:**

```
CLI Args (Viper) \                                                     / CLI (Table/CSV/etc.)
                  → Validation → Git Analysis → Scoring → Ranking → Output
MCP Request (URN) /                                                     \ MCP (JSON Response)
```

Hotspot can run as an MCP server (`hotspot mcp`) to expose its analysis capabilities as JSON-RPC tools with full parameter parity. **Critically, all MCP tools now support an optional `urn` parameter** to enable portable repository identity across machines (see Repository URN pattern below).

Agents should provide a `urn` to ensure analysis runs for the same repository are unified in the database, regardless of the local clone path. Note that `repo_path` (defaulting to `.`) is still required to perform fresh Git analysis.

## Self-Discovery & Guided Playbooks

Agents can autonomously discover context and workflows via MCP:

- **Resources**: `hotspot://docs/agents`, `hotspot://docs/metrics`.
- **Prompts**: `release-readiness` (Audit), `refactor-prioritization` (ROI).

## Core Domain Concepts

### Scoring Modes

The `core` package implements four distinct scoring algorithms based on different risk assessment principles. This is the most critical domain knowledge:

1. **Hot Mode** (Activity hotspots)
   - **Principle**: Identifies files with high recent activity and volatility.
   - **Focus**: Recent commits, churn, and active development.
   - **Use Case**: Find files currently undergoing active development or significant refactoring.

2. **Risk Mode** (Knowledge risk / bus factor)
   - **Principle**: Identifies files with concentrated ownership and high bus factor risk.
   - **Focus**: Few contributors, uneven ownership distribution, knowledge silos.
   - **Use Case**: Find files that would be problematic to maintain if key contributors leave.

3. **Complexity Mode** (Technical debt candidates)
   - **Principle**: Identifies large, old files with high maintenance burden.
   - **Focus**: File size, age, complexity, and historical churn.
   - **Use Case**: Find files that are expensive to modify or maintain.

4. **ROI Mode** (Refactoring priority)
   - **Principle**: Identifies files where refactoring effort provides the highest technical return.
   - **Focus**: High churn on complex/large legacy files (Technical impact vs. Effort).
   - **Use Case**: Prioritize refactoring targets in a large codebase with limited resources.

### Composite Modes (v1.22.0+)

Three composite modes blend two base algorithms to surface multi-dimensional risk. Agents should prefer these over base modes when the use case spans more than one risk dimension:

| Mode | Blend | When to Use |
|------|-------|-------------|
| **active_owners** | Hot (50%) + Risk (50%) | File is actively changing AND siloed — prioritize knowledge transfer. |
| **refactor_now** | Complexity (60%) + ROI (40%) | Sprint planning — rank files by highest refactoring return. |
| **legacy_debt** | Complexity (70%) + Risk (30%) | Pre-change audit — identify fragile, concentrated legacy systems. |

Composite scores, breakdowns, and reasoning for all base modes are returned together in a single MCP response, giving agents full signal transparency without additional round-trips.

## Repository Shape & Preset System

Hotspot includes **shape analysis** (lightweight single-pass aggregation) to characterize repositories and recommend presets.

**Three fixed presets:**
| Preset | Mode | Use Case |
|--------|------|----------|
| **small** | hot | CLI tools, microservices, libraries |
| **large** | roi | Large monorepos with deep histories |
| **infra** | risk | Infrastructure-as-code repositories |

**Workflow:** `hotspot init` (or `hotspot shape`) → review recommendation → apply preset to analysis.

## Key Design Patterns

- **I/O Caching**: Results and analysis are cached using pluggable backends (SQLite, MySQL, PostgreSQL) to dramatically speed up repeated analyses. Stores use a "light constructor" pattern where connection is established first, followed by an explicit `Initialize()` call for schema setup. See `internal/iocache/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huangsam/hotspot](https://github.com/huangsam/hotspot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
