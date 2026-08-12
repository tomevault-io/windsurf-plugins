---
trigger: always_on
description: - **Framework:** Next.js
---

# AGENTS.md

## Stack

### Frontend

- **Framework:** Next.js
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **UI Components:** shadcn/ui
- **Forms:** React Hook Form
- **Validation:** Zod
- **Draft Editor:** TipTap or Lexical
- **Server State / Data Fetching:** TanStack Query or Fetch API

### Backend API

- **Framework:** NestJS
- **Language:** TypeScript
- **ORM:** Prisma
- **API Style:** REST API for MVP
- **Validation:** Zod or class-validator
- **Authentication:** NextAuth/Auth.js, Clerk, or custom JWT
- **Authorization:** Role-Based Access Control for Teacher/Admin permissions
- **File Upload:** NestJS upload module / Multer
- **Workflow Client:** Temporal Client SDK

### Workflow Orchestration

- **Workflow Engine:** Temporal
- **Local Development:** Temporal Docker Compose
- **Workflow Monitoring:** Temporal Web UI
- **Main Workflows:** `CreateDraftExamWorkflow`, `MixExamWorkflow`, `PublishExamWorkflow`

### AI / Document Worker

- **Runtime:** Python
- **Worker SDK:** Temporal Python SDK
- **DOCX Parsing:** python-docx
- **XML Processing:** lxml
- **DOCX Fallback:** Mammoth, if needed
- **Data Validation:** Pydantic
- **AI Client:** OpenAI SDK through an AI Provider Interface

### Database

- **Primary Database:** PostgreSQL
- **Flexible Data Storage:** JSONB
- **Vector Extension:** pgvector
- **Keyword Search:** PostgreSQL Full-Text Search
- **Semantic Search:** pgvector similarity search
- **Hybrid Search:** Full-text search + vector search

### Vector / RAG Layer

- **Vector Store:** PostgreSQL + pgvector
- **Embedding Model:** OpenAI `text-embedding-3-small` or equivalent
- **Chunking:** Custom Python chunker based on Canonical Document blocks
- **Metadata Filtering:** PostgreSQL columns + JSONB metadata
- **Retrieval Strategy:** Metadata-filtered semantic retrieval, with hybrid search where needed
- **Content Hashing:** SHA-256 or equivalent to avoid unnecessary re-embedding

### File Storage

- **Object Storage:** Cloudflare R2
- **Local Development Storage:** MinIO or local filesystem
- **Stored Assets:** Original DOCX files, extracted images, generated DOCX files, answer sheets, matrices, and ZIP packages
- **Download Access:** Signed URLs

### Document Publishing

- **DOCX Rendering:** python-docx or docxtemplater
- **Answer Matrix Rendering:** HTML/DOCX/table renderer
- **ZIP Packaging:** Python zipfile or Node archiver
- **PDF Conversion:** LibreOffice Headless, optional Phase 2

### DevOps / Infrastructure

- **Containerization:** Docker
- **Local Environment:** Docker Compose
- **Deployment Target:** Railway, Render, Fly.io, Cloud Run, or VPS
- **Database Hosting:** Supabase, Neon, Railway Postgres, or managed PostgreSQL with pgvector support
- **CI/CD:** GitHub Actions
- **DNS / SSL:** Cloudflare
- **Secrets:** Environment variables or platform secrets

### Monitoring and Testing

- **Application Logs:** NestJS Logger or Pino
- **Worker Logs:** Python logging or structlog
- **Workflow Monitoring:** Temporal Web UI
- **Error Tracking:** Sentry
- **Backend Tests:** Jest
- **Frontend Tests:** Playwright / Testing Library
- **Python Tests:** Pytest
- **API Testing:** Postman or Bruno

---
This repository uses **Spec-Driven Development (SDD)**.

All product behavior must be specified before implementation. Specifications are the source of truth for development, testing, and review.

The repository has five named agents:

1. **Orion — Spec Architect**: creates and maintains specifications
2. **Lyra — UX Designer**: clarifies user experience decisions and updates UX-related specification content
3. **Vega — Solution Architect**: answers solution and architecture questions without modifying files
4. **Nova — Software Engineer**: implements approved specifications
5. **Pulsar — Review Agent**: validates specification compliance and implementation quality

Each agent has a strict role boundary and must not exceed it.

---

## Agent Invocation

The repository provides one Codex Skill per named agent under:

```text
/.agents/skills/<agent-name>/SKILL.md
```

Use the following Codex-native invocations:

| Agent | Role | Invocation |
|---|---|---|
| Orion | Spec Architect | `$orion` or `/skills` → `orion` |
| Lyra | UX Designer | `$lyra` or `/skills` → `lyra` |
| Vega | Solution Architect | `$vega` or `/skills` → `vega` |
| Nova | Software Engineer | `$nova` or `/skills` → `nova` |
| Pulsar | Review Agent | `$pulsar` or `/skills` → `pulsar` |

An invocation selects exactly one role for the current task. The selected agent must follow this file, `/specs/constitution.md`, and its own `SKILL.md`.

> Codex does not currently support repository-defined commands with arbitrary root names such as `/orion`. Repository-scoped Skills are the supported shared mechanism; use `$orion` or select it through `/skills`.

---

## Repository Specification Model

### Global Constitution

The global product and architecture rules are defined in:

```text
/specs/constitution.md
```

The constitution is the highest-level specification document. It defines the product principles, architecture direction, naming conventions, quality standards, workflow rules, and non-negotiable constraints.

All feature specifications must comply with the constitution.

### Feature Specification Folders


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [knt-work/codians](https://github.com/knt-work/codians) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
