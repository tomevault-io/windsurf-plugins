---
trigger: always_on
description: > **For AI coding assistants**: This document provides essential context about the Consult repository's architecture, intent, and organization.
---

# Consult Repository Guide

> **For AI coding assistants**: This document provides essential context about the Consult repository's architecture, intent, and organization.

## Pull Requests

Always use the PR template at `.github/PULL_REQUEST_TEMPLATE.md` when writing PR descriptions.

---

## Project Overview

**Consult** combines AI with human oversight to process public consultation responses at scale for UK government policy-making.

**Core workflow**:
1. Upload citizen responses to government consultations
2. AI automatically identifies themes across responses
3. Humans review, edit, and approve themes
4. AI assigns finalized themes to individual responses
5. Interactive dashboard for policy analysis

**Status**: Incubation project trialling across UK Civil Service  
**Repository**: `i-dot-ai/consult`

---

## Technology Stack

**Backend**: Python 3.12, Django 5.2.11, PostgreSQL with pgvector, Redis, Django RQ  
**Frontend**: Astro 5.16.6, Svelte, Tailwind CSS, TypeScript
**Infrastructure**: AWS ECS, AWS Batch, Lambda, S3, Terraform  
**AI Library**: `themefinder` 0.8.2 (theme discovery and assignment)  
**LLM**: Azure OpenAI (text-embedding-3-large for embeddings)

---

## Architecture

### High-Level Structure

```
Frontend (Astro/Svelte) ←→ Backend API (Django REST) ←→ PostgreSQL
                                      ↓
                        AWS Batch Jobs (Themefinder)
                                      ↓
                        S3 (data interchange)
                                      ↓
                        Lambda → RQ Jobs → Import results
```

### Project Structure

```
consult/
├── backend/                      # Django REST API
│   ├── consultations/            # Core domain models & views
│   ├── data_pipeline/            # Batch job orchestration & S3 sync
│   ├── authentication/           # JWT + OIDC auth
│   └── ingest/                   # Data import/deletion
│
├── frontend/                     # Astro + Svelte
│   └── src/
│       ├── components/           # Svelte components
│       ├── pages/                # Astro pages (routes)
│       └── middleware.ts         # Auth & security
│
├── pipeline-sign-off/            # AWS Batch: Find themes
├── pipeline-mapping/             # AWS Batch: Assign themes
├── lambda/                       # Event handlers (import results)
├── terraform/                    # Infrastructure as Code
└── docs/architecture/decisions/  # ADRs
```

---

## Core Domain Model

**Key entities** (see `backend/consultations/models.py`):

```
Consultation (title, stage, code)
  └─> Question (text, type: open/closed/demographic)
       ├─> Response (text, embedding[vector], search_vector)
       │    ├─> Respondent (demographics)
       │    └─> ResponseAnnotation (AI analysis: themes, sentiment)
       ├─> CandidateTheme (AI-generated, pre-approval)
       └─> SelectedTheme (human-approved, final)
```

**Key characteristics**:
- **UUIDs** as primary keys throughout
- **Denormalized response storage**: All response types in `Response.text` (see ADR-0006)
- **Vector embeddings**: 3072-dim pgvector for semantic search
- **Dual themes**: Separate CandidateTheme (AI) vs SelectedTheme (human-approved)
- **Audit trail**: Through model `ResponseAnnotationTheme` tracks AI vs human assignments

---

## Data Pipeline (Event-Driven)

### Consultation Stages

```
SETUP → FINDING_THEMES → FINALISING_THEMES → ASSIGNING_THEMES → ANALYSIS
```

### Pipeline Flow

**1. Setup** (`Consultation.Stage.SETUP`)
- Upload CSV/Excel → Parse → Load to PostgreSQL
- Generate embeddings for free-text responses
- Location: `backend/ingest/`, `backend/data_pipeline/jobs.py::import_consultation`

**2. Finding Themes** (`FINDING_THEMES`)
- Export data to S3 → AWS Batch job → Themefinder generates themes → S3
- EventBridge → Lambda → RQ job imports `CandidateTheme` records
- Location: `pipeline-sign-off/find_themes_script.py`, `lambda/import_candidate_themes/`

**3. Finalising Themes** (`FINALISING_THEMES`)
- Users review/edit candidate themes in UI
- Approve themes → create `SelectedTheme` records
- Location: `frontend/src/pages/consultations/[id]/questions/[qid]/themes/`

**4. Assigning Themes** (`ASSIGNING_THEMES`)
- Export selected themes to S3 → AWS Batch job → Assign themes to responses → S3
- EventBridge → Lambda → RQ job imports `ResponseAnnotation` records
- Location: `pipeline-mapping/assign_themes_script.py`, `lambda/import_response_annotations/`

**5. Analysis** (`ANALYSIS`)
- Dashboard shows theme distribution, filtered responses, sentiment
- Location: `frontend/src/pages/consultations/[id]/analysis/`

### S3 Data Contract

```
app_data/consultations/{code}/
├── inputs/
│   └── question_{id}/
│       ├── question.json
│       ├── responses.jsonl
│       └── themes.csv (for assign phase)
└── outputs/
    ├── find-themes/{timestamp}/
    │   └── question_{id}/candidate_themes.csv
    └── assign-themes/{timestamp}/
        └── question_{id}/response_annotations.jsonl
```

---

## Key Architectural Patterns

### 1. Event-Driven Pipeline
**Why**: Decouple batch processing, scale independently, fault-tolerant  
**How**: Django → AWS Batch → EventBridge → Lambda → RQ → PostgreSQL  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [i-dot-ai/consult](https://github.com/i-dot-ai/consult) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
