---
trigger: always_on
description: Generates User Requirements Specifications by querying Pinecone for relevant GAMP 5 guidance.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CSV-GameChanger is a GAMP 5 and CSA (Computer Software Assurance) compliant CSV (Computer System Validation) Engine.

## Branding Convention (IMPORTANT — follow exactly)

This project uses **dual branding**. Getting this wrong causes rework.

| Concept | Name | Where to use |
|---------|------|-------------|
| **Product / tool brand** | **EVOLV** (all caps) | "Powered by" lines, tool-name contexts, sidebar logo text, legal/signature text |
| **Functional label** | **Validation Factory** | Page headers / hero sections, navigation menu, about text, process descriptions, requirement-generation logic |
| **Combined form** | **EVOLV: The Validation Factory** or **EVOLV \| The Validation Factory** | Browser tab title, PDF headers, CLI banners |
| **Company** | **WingstarTech Inc.** | Footer attribution |

**Rules:**
1. The main page header (hero) for Tab 6 must say **"Validation Factory"**, NOT "EVOLV".
2. The sidebar nav label for Tab 6 must say **"6. Validation Factory"**.
3. Footer stays: `"Powered by EVOLV | A WingstarTech Inc. Product"`.
4. PDF headers use: `"EVOLV | The Validation Factory"`.
5. Sidebar logo: "EVOLV" with subtitle "THE VALIDATION FACTORY".
6. **Never** replace "Validation Factory" with just "EVOLV" in user-facing text — "Validation Factory" is the core business identity.
7. Old names ("Trustme AI", "CSV Engine") are retired. Internal class names like `CSVEngineError` are kept to avoid breaking imports.

## Build and Development Commands

```bash
# Install dependencies
pip install fastapi uvicorn pydantic pinecone openai langchain-community langchain-text-splitters fpdf2

# Run the API server
uvicorn API.main:app --reload

# Run from project root
cd C:\Users\sreej\OneDrive\Desktop\CSV-GameChanger
uvicorn API.main:app --reload --host 0.0.0.0 --port 8000

# Generate URS document (interactive mode)
python scripts/draft_urs.py

# Generate URS from file
python scripts/draft_urs.py -f requirements.txt -n "Project Name"
```

## Architecture

```
CSV-GameChanger/
├── Agents/
│   ├── __init__.py
│   ├── risk_strategist.py       # GAMP 5 risk assessment logic
│   ├── requirement_architect.py # URS generation from natural language
│   ├── verification_agent.py    # URS verification against GAMP 5 text
│   └── integrity_manager.py     # Central audit trail + logic archives
├── API/
│   ├── __init__.py
│   └── main.py                  # FastAPI app with ServiceNow webhook
├── scripts/
│   ├── setup_pinecone_index.py  # Creates Pinecone index
│   ├── ingest_docs.py           # Ingests GAMP 5 PDFs to Pinecone
│   └── draft_urs.py             # Generate URS documents from requirements
├── utils/
│   ├── __init__.py
│   └── pdf_generator.py         # URS PDF export with signature page
├── output/
│   ├── urs/                     # Generated URS Markdown/PDF files
│   └── logic_archives/          # Hidden JSON logic-archive files (generated)
├── audit_trail.log              # 21 CFR Part 11 compliant audit log (generated)
└── CLAUDE.md
```

## Current Implementation State

### API/main.py

**Endpoint:** `POST /webhook/sn-change`

Receives ServiceNow Change Requests and triggers automated risk assessment.

**Request Model (`ServiceNowChangeRequest`):**
```python
{
    "cr_id": str,              # Change Request ID
    "description": str,         # Change description
    "system_criticality": str,  # "high", "medium", "low", "critical", "minor"
    "change_type": str          # "emergency", "normal", "standard", "routine"
}
```

**Response Model (`ChangeRequestResponse`):**
```python
{
    "status": "assessed",
    "cr_id": str,
    "message": str,
    "timestamp": str,
    "risk_assessment": {
        "severity": str,           # HIGH, MEDIUM, LOW
        "occurrence": str,         # FREQUENT, OCCASIONAL, RARE
        "detectability": str,      # HIGH, MEDIUM, LOW
        "rpn": int,                # Risk Priority Number (1-27)
        "risk_level": str,         # "High", "Medium", "Low"
        "testing_strategy": str,   # CSA recommendation
        "patient_safety_override": bool
    }
}
```

**Audit Events Logged:**
1. `CHANGE_REQUEST_RECEIVED` - When CR arrives
2. `RISK_ASSESSMENT_COMPLETED` - After risk calculation
3. `CHANGE_REQUEST_FAILED` - On any error

**Exception Classes:**
- `CSVEngineError` - Base exception
- `ValidationError` (CSV-001) - Input validation failed
- `AuditLogError` (CSV-002) - Audit logging failed
- `ProcessingError` (CSV-003) - Processing failed

### Agents/risk_strategist.py

**Risk Strategist Agent** - Implements GAMP 5 risk-based approach.

**Enums:**
- `RiskLevel`: LOW, MEDIUM, HIGH
- `Severity`: LOW (1), MEDIUM (2), HIGH (3)
- `Occurrence`: RARE (1), OCCASIONAL (2), FREQUENT (3)
- `Detectability`: HIGH (1), MEDIUM (2), LOW (3)
- `TestingStrategy`: UNSCRIPTED, HYBRID, RIGOROUS_SCRIPTED

**Core Functions:**

| Function | Input | Output | Purpose |
|----------|-------|--------|---------|
| `calculate_risk_score()` | Severity, Occurrence, Detectability | (RPN, RiskLevel) | Calculates Risk Priority Number |
| `get_csa_testing_strategy()` | RiskLevel | TestingStrategy | Returns CSA testing recommendation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sreejisworld/CSV-GameChanger](https://github.com/sreejisworld/CSV-GameChanger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
