---
trigger: always_on
description: Generates 3 embedding types: `jd_summary`, `required_skills`, `responsibilities`. Runs in background.
---

# TranspaHire – FastAPI LLM & Embedding Microservice

> **Maintenance note:** Update only the relevant section when code changes. Do not rewrite the full document.

---

## 1. Project Overview

### Purpose
A Python microservice within the TranspaHire platform. The NestJS backend handles core business logic; this service handles all AI-heavy work:

- **Resume parsing** – extract structured candidate data from uploaded files using LLMs
- **Embedding generation** – produce semantic vectors for candidates and jobs to power downstream matching

### Core Responsibilities
- Accept file uploads (PDF, DOCX, images) and return structured `ParsedResumeData` JSON
- Generate local embeddings (sentence-transformers, no API key required)
- Generate multi-vector embeddings (Gemini API) for candidates and jobs, stored in pgvector
- Validate JWT tokens issued by the NestJS service for protected routes

### High-level Architecture
```
Client / NestJS
      │
      ▼
FastAPI (this service)
      │
      ├── /resumes/parse-resume        → Unstructured → Gemini / OpenAI / HF / regex
      ├── /embeddings/generate-*       → sentence-transformers (local, no auth)
      └── /embeddings/candidates|jobs  → Gemini API → pgvector DB (JWT required)
```

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (async) |
| Language | Python 3.10+ |
| Primary LLM | Google Gemini (`gemini-1.5-flash`) |
| Embedding (remote) | Gemini `gemini-embedding-001` (768-dim) |
| Embedding (local) | `sentence-transformers` / `intfloat/e5-base-v2` (768-dim) |
| Document extraction | `unstructured` (primary), PyPDF2 / python-docx / pytesseract (legacy fallbacks) |
| Vector DB | PostgreSQL + `pgvector` extension |
| ORM | SQLAlchemy async (`asyncpg` driver) |
| Image preprocessing | Pillow |
| Auth | PyJWT (HS256, validates tokens from NestJS) |
| Rate limiting | `slowapi` |
| Config | `pydantic-settings` |
| CORS / GZip | FastAPI middleware |

---

## 3. Folder Structure

```
app/
├── main.py                  # App factory, router registration, middleware setup
├── config.py                # Settings loaded from .env via pydantic-settings
├── database.py              # Async SQLAlchemy engine + session factory (DatabaseManager)
├── dependencies.py          # JWT auth (get_current_user), subscription guard
│
├── routers/                 # FastAPI route handlers (thin – delegate to services)
│   ├── health.py            # GET /health
│   ├── resumes.py           # POST /resumes/parse-resume
│   ├── embeddings_local.py  # POST /embeddings/generate-embedding|generate-batch-embeddings
│   └── embeddings.py        # POST|GET /embeddings/candidates|jobs (auth required)
│
├── services/                # Business logic
│   ├── file_service.py      # ResumeParserService + UnstructuredExtractor + FileService
│   ├── gemini_service.py    # Gemini LLM – resume parsing (primary)
│   ├── openai_service.py    # OpenAI LLM – resume parsing (first fallback)
│   ├── huggingface_service.py  # HuggingFace LLM – resume parsing (second fallback)
│   ├── unstructured_service.py # Standalone unstructured wrapper (unused directly)
│   ├── embedding_service_local.py    # Local sentence-transformer embeddings (singleton)
│   ├── multi_vector_embedding_service.py  # Gemini multi-vector embeddings (DB-backed)
│   ├── gemini_service_embedding.py   # Gemini embedding API client
│   └── representation_service.py    # Text generation for each embedding type
│
├── models/                  # Data models
│   ├── schemas.py           # Pydantic request/response schemas
│   ├── embedding_models.py  # SQLAlchemy ORM: CandidateEmbedding, JobEmbedding
│   ├── database_models.py   # SQLAlchemy ORM: Profile, WorkExperience, Education, etc.
│   └── enums.py             # ParseStatus, ProficiencyLevel, SkillSource, Role, etc.
│
├── crud/
│   └── resume_crud.py       # ProfileCRUD, WorkExperienceCRUD, EducationCRUD, ProfileSkillCRUD
│
├── middleware/
│   └── errorhandler.py      # ErrorHandlerMiddleware + setup_exception_handlers
│
└── utils/
    ├── file_utils.py        # validate_upload() – checks file type & size
    ├── json_utils.py        # JSON helpers
    └── text_utils.py        # Text processing helpers
```

---

## 4. API Documentation

Base prefix: `/api/v1`

---

### `GET /health`
- **Auth:** None
- **Response:** `{"status": "ok"}`

---

### `POST /resumes/parse-resume`
- **Auth:** None
- **Content-Type:** `multipart/form-data`

**Request fields:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `file` | File | Yes | – | Resume file (PDF, DOCX, DOC, JPG, PNG, TIFF, BMP) |
| `resume_id` | int | Yes | – | ID assigned by caller (returned in response for correlation) |
| `enhance_images` | bool | No | `true` | Apply OCR preprocessing (contrast/sharpness boost) |

**Response: `ParseResponse`**
```json
{
  "success": true,
  "resume_id": 42,
  "parsed_data": {
    "personal_info": { "name": "...", "email": "...", "phone": "...", "linkedin": "...", "github": "..." },
    "summary": "...",
    "skills": {
      "technical_skills": [{ "name": "Python", "type": "TECHNICAL", "group": "Programming", ... }],
      "soft_skills": [{ "name": "Communication", ... }]
    },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YuvarajD1998) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
