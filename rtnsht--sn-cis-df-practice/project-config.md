---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This folder contains study materials for the **ServiceNow CIS-DF (Certified Implementation Specialist - Data Foundations)** certification exam covering CMDB and CSDM.

## Folder Structure

```
CIS - Data Foundation - Learning and Practice/
├── docs/                              # Documentation & study materials
│   ├── servicenow/
│   │   ├── pdf/                       # Original ServiceNow PDFs
│   │   └── markdown/                  # AI-readable markdown versions
│   ├── courses/                       # LinkedIn Learning courses
│   ├── ServiceNow CIS-DF...pdf        # Official exam blueprint
│   ├── INTERACTIVE_QUIZ_GUIDE.md      # Quiz system documentation
│   └── requirements.md                # Project requirements
│
├── practice/                          # Practice exam system
│   ├── app/                           # Web app (exam HTML files)
│   ├── results/                       # Completed quiz results
│   └── config/                        # Configuration files
│
├── tools/                             # Utility scripts
│   ├── sync-history.js                # Question history management
│   └── pdf_to_markdown.py             # PDF conversion utility
│
├── CLAUDE.md                          # AI instructions (this file)
└── README.md                          # Project overview
```

## Exam Blueprint

**Exam Code:** CIS-DF | **Questions:** 75 | **Duration:** 90 minutes

| Domain | Weight | Key Topics |
|--------|--------|------------|
| 1. Configuration | 15% | CI Class Manager, IRE (Identification & Reconciliation Engine), CMDB 360/multisource |
| 2. Ingest | 19% | Auto vs manual relationships, Service Graph Connectors, data ingestion methods, technical debt |
| 3. Govern | 35% | CMDB Health Score (6 metrics), Data Manager, deduplication, CMDB Workspace, Principal Classes, CI lifecycle |
| 4. Insight | 20% | NLQ, CMDB 360 reports, Unified Map, dependency views, Playbooks |
| 5. CSDM Fundamentals | 11% | CI mapping to CSDM domains, stakeholder collaboration, CSDM benefits |

## Question Types

Generate questions in these formats (matching the real exam):
- **Multiple Choice** - Single correct answer (A, B, C, D)
- **Multiple Select** - "Choose two/three" with multiple correct answers
- **Matching** - Pair terms with definitions or put items in sequence
- **Scenario-Based** - Real-world situations requiring applied knowledge

## Resources

- `docs/ServiceNow CIS-DF Exam Specification_Blueprint.pdf` - Official exam blueprint with sample questions
- `docs/servicenow/markdown/` - 70+ ServiceNow Zurich release documentation markdown files covering CMDB topics. Use this for AI reading.
- `docs/servicenow/pdf/` - 70+ ServiceNow Zurich release documentation PDFs. Fall back if markdown files have issues.

## How to Generate Practice Questions

1. **Read relevant files** from `docs/servicenow/markdown/` folder based on the topic
2. **Create scenario-based questions** that test practical application, not just memorization
3. **Include answer explanations** with references to concepts from documentation
4. **Weight question distribution** according to domain percentages (Govern 35% = most questions)

## Key Concepts by Domain

### Configuration (15%)
- CI Class Manager: table structure, class attributes, hierarchy
- IRE: identification rules, reconciliation rules, duplicate detection
- CMDB 360: multisource data aggregation, source precedence

### Ingest (19%)
- Service Graph Connectors (AWS, Azure, GCP, SCCM, Intune, etc.)
- IntegrationHub ETL
- Discovery vs manual CI creation
- Asset-CI synchronization (Install Status, Hardware Status sync automatically)

### Govern (35%)
- **Health Score 6 Metrics:** Correctness, Completeness, Compliance, Relationship, Staleness, Duplicate
- Data Manager policies and requirements
- Deduplication Wizard
- CMDB Workspace features
- Principal Classes definition
- CI lifecycle attributes (operational status, install status)

### Insight (20%)
- Natural Language Query (NLQ)
- CMDB Query Builder
- Unified Map configuration and usage
- Dependency Views
- Data Foundations Dashboard Playbooks structure:
  1. Summary of indicator
  2. Overview of problem
  3. Importance of addressing issue
  4. Fix or Improve

### CSDM Fundamentals (11%)
- CSDM domains and layers
- Mapping CIs to correct CSDM classes
- Business Services, Technical Services, Application Services
- Stakeholder roles in CSDM adoption

## Sample Question Format

```
**Question X (Domain: [Domain Name])**
[Scenario or question text]

A. [Option A]
B. [Option B]
C. [Option C]
D. [Option D]

<details>
<summary>Answer</summary>
**Answer: [Letter]**
[Explanation of why this is correct and why others are wrong]
</details>
```

## Session Commands

When user asks to practice, offer these options:

### Topic Quiz (Learning Mode) - Interactive
- `"Quiz me on Configuration"` - 7-10 questions, untimed, with explanations
- `"Quiz me on Ingest"` - 7-10 questions on data ingestion
- `"Quiz me on Govern"` - 10-15 questions (largest domain)
- `"Quiz me on Insight"` - 7-10 questions on reporting/visualization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rtnsht/sn-cis-df-practice](https://github.com/rtnsht/sn-cis-df-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
