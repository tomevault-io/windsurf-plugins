---
trigger: always_on
description: An educational simulation platform where students roleplay as biomedical data analysts, interacting with LLM-powered "Stakeholders" based on real datasets. Uses a sophisticated **meta-prompt architecture** where an Architect LLM generates custom scenarios and Actor system prompts for dynamic, dataset-aware conversations.
---

# Biomedical Analyst Roleplay Platform

## Project Overview

An educational simulation platform where students roleplay as biomedical data analysts, interacting with LLM-powered "Stakeholders" based on real datasets. Uses a sophisticated **meta-prompt architecture** where an Architect LLM generates custom scenarios and Actor system prompts for dynamic, dataset-aware conversations.

## Architecture

### Tech Stack
- **Backend**: Python Flask API (serverless-ready for Vercel)
- **Frontend**: React + Vite SPA
- **Database**: Supabase (PostgreSQL)
- **LLM**: Google Gemini (gemini-1.5-flash)
- **Deployment**: Vercel (monolithic)

### Meta-Prompt Architecture

**Two-Stage LLM System:**

1. **Architect LLM** (Stage 1): Generates comprehensive scenarios
   - Input: Dataset metadata (columns, sample data, quality notes)
   - Output: Structured JSON with scenario details + custom Actor system prompt
   - Purpose: Creates unique, educational scenarios tailored to each dataset

2. **Actor LLM** (Stage 2): Roleplays as stakeholder
   - Input: Architect-generated system prompt + chat history
   - Output: In-character responses that guide students
   - Purpose: Provides business context, refuses to write code, teaches data cleaning

### Key Innovation

Instead of hardcoded chat prompts, the Architect LLM **generates custom system prompts** for the Actor that:
- Know specific column names from the dataset
- Mention data quality issues naturally
- Guide students toward data cleaning challenges
- Stay in character and refuse to write code

## Project Structure

```
tubes-pdb/
├── api/
│   └── index.py                 # Vercel serverless entry point
├── backend/
│   ├── app.py                   # Flask app factory
│   ├── config.py                # Environment configuration
│   ├── models/                  # Pydantic data models
│   │   ├── user.py             # Lecturer model
│   │   ├── student.py          # Student model
│   │   ├── dataset.py          # Dataset with meta-prompt fields
│   │   ├── assignment.py       # Scenario model
│   │   ├── chat_message.py     # Chat history
│   │   ├── submission.py       # Student submissions
│   │   └── grade.py            # Grading
│   ├── routes/                  # API endpoints (blueprints)
│   │   ├── auth.py             # Authentication
│   │   ├── datasets.py         # Dataset CRUD
│   │   ├── assignments.py      # JIT scenario generation
│   │   ├── chat.py             # Chat with Actor LLM
│   │   ├── submissions.py      # Submission handling
│   │   └── grading.py          # Grading interface
│   ├── services/                # Business logic
│   │   ├── llm_service.py      # Gemini integration (Architect + Actor)
│   │   ├── auth_service.py     # JWT + bcrypt
│   │   └── assignment_service.py # JIT generation logic
│   └── utils/
│       ├── db.py               # Supabase client
│       └── validators.py       # Input validation
├── frontend/
│   └── src/
│       ├── pages/
│       │   ├── student/        # Student UI
│       │   └── lecturer/       # Lecturer UI
│       ├── context/            # React context (auth)
│       ├── services/           # API client (axios)
│       └── index.css           # Design system
├── docs/
│   ├── prd.md                  # Product requirements
│   ├── meta_prompt.md          # Meta-prompt architecture
│   ├── example_dataset.md      # Dataset examples
│   ├── db_schema.sql           # Database schema
│   └── vercel_deployment.md    # Deployment guide
├── requirements.txt            # Python dependencies
├── vercel.json                 # Vercel config (monolithic)
└── .env.example                # Environment template
```

## Database Schema

### Tables

**users** (Lecturers)
- `id` (UUID, PK)
- `email` (unique)
- `password_hash` (bcrypt)
- `created_at`

**students**
- `nim` (VARCHAR, PK) - Student ID
- `name`
- `created_at`

**datasets** (Enhanced for meta-prompt)
- `id` (UUID, PK)
- `name`
- `url` (Google Drive link)
- `metadata_summary`
- `columns_list` (TEXT[]) - Column names
- `sample_data` (TEXT) - CSV format
- `data_quality_notes` (TEXT) - Known issues
- `created_at`

**assignments**
- `id` (UUID, PK)
- `student_nim` (FK → students, unique)
- `dataset_id` (FK → datasets)
- `scenario_json` (JSONB) - Architect output
- `created_at`

**chat_messages**
- `id` (UUID, PK)
- `assignment_id` (FK → assignments)
- `sender` ('student' | 'ai')
- `content` (TEXT)
- `timestamp`

**submissions**
- `id` (UUID, PK)
- `assignment_id` (FK → assignments)
- `link_url` (Google Drive/Colab)
- `submission_type` ('progress' | 'final')
- `created_at`

**grades**
- `assignment_id` (UUID, PK, FK → assignments)
- `score` (0-100)
- `feedback` (TEXT)
- `created_at`

## API Endpoints

### Authentication
- `POST /api/auth/student/login` - NIM-only login
- `POST /api/auth/lecturer/login` - Email/password
- `POST /api/auth/lecturer/register` - Create lecturer
- `POST /api/auth/students/upload-roster` - Bulk upload (CSV)

### Datasets
- `GET /api/datasets` - List all (lecturer only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IritaSee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
