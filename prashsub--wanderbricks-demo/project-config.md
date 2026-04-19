---
trigger: always_on
description: Comprehensive methodology for creating multi-phase project plans for Databricks data platform solutions
---


# Project Plan Methodology for Databricks Solutions

## Pattern Recognition

This rule documents the comprehensive methodology for creating multi-phase project plans for Databricks data platform solutions. The patterns were discovered and refined during the Databricks Health Monitor project planning process.

**Key Assumption:** Planning starts AFTER Bronze ingestion and Gold layer design are complete. These are prerequisites, not phases.

---

## When to Apply This Rule

Apply this methodology when:
- Creating architectural plans for Databricks data platform projects
- Building observability/monitoring solutions using system tables
- Planning multi-artifact solutions (TVFs, Metric Views, Dashboards, etc.)
- Developing agent-based frameworks for platform management
- Creating frontend applications for data platform interaction

---

## Plan Structure Framework

### Prerequisites (Not Numbered Phases)

Before planning begins, these must be complete:

| Prerequisite | Description | Status |
|--------------|-------------|--------|
| Bronze Layer | Raw data ingestion from source systems | ✅ Complete |
| Silver Layer | DLT streaming with data quality | ✅ Complete |
| Gold Layer | Dimensional model (star schema) | ✅ Complete |

### Standard Project Phases

```
plans/
├── README.md                              # Index and overview
├── prerequisites.md                       # Bronze/Silver/Gold summary (optional)
├── phase1-use-cases.md                    # Analytics artifacts (master)
│   ├── phase1-addendum-1.1-ml-models.md   # Machine Learning
│   ├── phase1-addendum-1.2-tvfs.md        # Table-Valued Functions
│   ├── phase1-addendum-1.3-metric-views.md # UC Metric Views
│   ├── phase1-addendum-1.4-lakehouse-monitoring.md # Monitoring
│   ├── phase1-addendum-1.5-ai-bi-dashboards.md # Dashboards
│   ├── phase1-addendum-1.6-genie-spaces.md # Natural Language
│   └── phase1-addendum-1.7-alerting-framework.md # Alerting
├── phase2-agent-framework.md              # AI Agents
└── phase3-frontend-app.md                 # User Interface
```

### Phase Dependencies

```
Prerequisites (Bronze → Silver → Gold) → Phase 1 (Use Cases) → Phase 2 (Agents) → Phase 3 (Frontend)
         [COMPLETE]                               ↓
                                           All Addendums
```

---

## Agent Domain Framework

### Core Principle

**ALL artifacts across ALL phases MUST be organized by Agent Domain.** This ensures:
- Consistent categorization across 100+ artifacts
- Clear ownership by future AI agents
- Easy discoverability for users
- Aligned tooling for each domain

### Standard Agent Domains

| Domain | Icon | Focus Area | Key Gold Tables |
|--------|------|------------|-----------------|
| **Cost** | 💰 | FinOps, budgets, chargeback | `fact_usage`, `dim_sku`, `commit_configurations` |
| **Security** | 🔒 | Access audit, compliance | `fact_audit_events`, `fact_table_lineage` |
| **Performance** | ⚡ | Query optimization, capacity | `fact_query_history`, `fact_node_timeline` |
| **Reliability** | 🔄 | Job health, SLAs | `fact_job_run_timeline`, `dim_job` |
| **Quality** | ✅ | Data quality, governance | `fact_data_quality_monitoring_table_results` |

### Agent Domain Application

Every artifact (TVF, Metric View, Dashboard, Alert, ML Model, Monitor, Genie Space) must:
1. Be tagged with its Agent Domain
2. Use the domain's Gold tables
3. Answer domain-specific questions
4. Be grouped with related domain artifacts in documentation

**Example Pattern:**

```markdown
## 💰 Cost Agent: get_top_cost_contributors

**Agent Domain:** 💰 Cost
**Gold Tables:** `fact_usage`, `dim_workspace`
**Business Questions:** "What are the top cost drivers?"
```

---

## Agent Layer Architecture Pattern

### Core Principle: Agents Use Genie Spaces as Query Interface

**AI Agents DO NOT query data assets directly.** Instead, they use Genie Spaces as their natural language query interface. Genie Spaces translate natural language to SQL and route to appropriate tools (TVFs, Metric Views, ML Models).

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     USERS (Natural Language)                             │
│  "Why did costs spike last Tuesday?"                                    │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              PHASE 2: AI AGENT LAYER (LangChain/LangGraph)              │
│                                                                          │
│  ┌────────────────────────────────────────────────────────┐            │
│  │ ORCHESTRATOR AGENT                                     │            │
│  │ • Intent classification (cost, security, performance)  │            │
│  │ • Routes to specialized agents                         │            │
│  │ • Coordinates multi-agent workflows                    │            │
│  │ • Uses: Unified Health Monitor Genie Space             │            │
│  └────────────────────────────────────────────────────────┘            │
│                               │                                          │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prashsub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
