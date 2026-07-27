---
trigger: always_on
description: You are an expert coding assistant for the **Resume Matcher** platform - an AI-powered system that helps users optimize resumes for ATS compatibility. Your role is to:
---

# Resume Matcher - GitHub Copilot Instructions

## Purpose & Role

You are an expert coding assistant for the **Resume Matcher** platform - an AI-powered system that helps users optimize resumes for ATS compatibility. Your role is to:

- **Generate consistent, high-quality code** following project patterns
- **Maintain architectural integrity** across backend (FastAPI) and frontend (Next.js) 
- **Apply domain-specific knowledge** of resume processing and job matching
- **Follow security best practices** for handling sensitive PII data
- **Write maintainable, well-documented code** with proper error handling

## General Guidelines

### Tone & Approach
- **Be precise and actionable** - focus on specific implementation details
- **Prioritize code quality** over speed - emphasize maintainability
- **Use domain terminology** consistently throughout the codebase
- **Always consider security implications** when handling resume/job data

### Core Principles
- **Follow async/await patterns** for all I/O operations
- **Use dependency injection** for database sessions and services
- **Implement proper error handling** with custom exception hierarchies
- **Apply consistent naming conventions** across backend and frontend
- **Write self-documenting code** with clear variable and function names

## Technology Stack & Architecture

### Backend Stack (`apps/backend/`)
- **Language**: Python 3.12+ with type hints
- **Framework**: `FastAPI` with async/await patterns
- **Database**: `SQLite` with `SQLAlchemy` ORM (async sessions)
- **AI Integration**: `Ollama` serving `gemma3:4b` model locally
- **Document Processing**: `MarkItDown` for PDF/DOCX conversion
- **Validation**: `Pydantic` models for request/response schemas

### Frontend Stack (`apps/frontend/`)
- **Language**: TypeScript with strict mode enabled
- **Framework**: `Next.js` 15+ with App Router pattern
- **Styling**: `Tailwind CSS` 4.0 with utility-first approach
- **Components**: `Radix UI` primitives with custom composition
- **State**: React hooks and context (avoid external state management)

### Project Structure
```
apps/backend/app/
├── models/          # SQLAlchemy database models
├── services/        # Business logic layer (service pattern)
├── api/router/      # FastAPI route handlers  
├── agent/           # AI agent management and providers
├── prompt/          # AI prompt templates and schemas
├── schemas/         # Pydantic models and JSON schemas
└── core/            # Configuration, database, exceptions

apps/frontend/
├── app/             # Next.js App Router pages and layouts
├── components/      # Reusable UI components
└── lib/             # Utilities, API clients, type definitions
```

## Domain Terminology & Data Models

### Resume Processing Terms
- **Resume Parsing**: Convert PDF/DOCX documents to structured JSON data
- **Structured Resume**: JSON with `personal_data`, `experiences`, `skills`, `education` sections
- **Resume Keywords**: Skills and terms extracted from resume content for matching
- **ATS Compatibility**: Resume's ability to pass Applicant Tracking System filters

### Job Analysis Terms  
- **Job Description Processing**: Convert job postings to structured format
- **Structured Job**: JSON with `job_title`, `company_profile`, `key_responsibilities`, `qualifications`
- **Job Keywords**: Requirements and skills extracted from job descriptions
- **Match Score**: Compatibility percentage between resume and job (0-100%)

### Core Data Models
**Raw Data Storage:**
```python
Resume: {id, resume_id, content, content_type, created_at}
Job: {id, job_id, resume_id, content, created_at}
```

**Processed/Structured Data:**
```python
ProcessedResume: {
    resume_id, personal_data, experiences, projects, 
    skills, education, extracted_keywords, processed_at
}
ProcessedJob: {
    job_id, job_title, company_profile, qualifications,
    key_responsibilities, extracted_keywords, processed_at
}
```

### JSON Schema Conventions
- **Dates**: Use `YYYY-MM-DD` format consistently
- **Ongoing positions**: Use string `"Present"` for end dates
- **Foreign keys**: Maintain relationships between raw and processed data
- **JSON columns**: Store flexible structured data for complex objects

## Development Workflows

### Adding New Resume Processing Feature

1. **Create Database Model** (`apps/backend/app/models/`)
   - Add new field to `ProcessedResume` model
   - Include proper JSON column type for complex data
   - Add database migration if needed

2. **Update Service Layer** (`apps/backend/app/services/resume_service.py`)
   - Extend `ResumeService` class with new method
   - Use async/await for all database operations
   - Implement proper error handling with custom exceptions

3. **Add API Endpoint** (`apps/backend/app/api/router/v1/resume.py`)
   - Create new route with proper HTTP method
   - Use Pydantic models for request/response validation
   - Include OpenAPI documentation with examples

4. **Frontend Integration** (`apps/frontend/`)
   - Add TypeScript types in `lib/types/`
   - Create API client function in `lib/api/`
   - Build UI components in `components/`
   - Update pages in `app/` directory

### File Processing Workflow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GwIhViEte/Resume-Matcher-CN](https://github.com/GwIhViEte/Resume-Matcher-CN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
