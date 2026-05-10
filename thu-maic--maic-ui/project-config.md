---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

Learn Your Way is a full-stack monorepo with a **well-structured backend** that needs **frontend integration**. The system transforms PDFs into personalized learning experiences across 5 modalities.

**Critical Context**: The backend API is comprehensive and functional, but the frontend operates entirely on static mock data. The primary development task is bridging this disconnect.

### Technology Stack
- **Backend**: python3  FastAPI, SQLAlchemy ORM, SQLite (dev) / PostgreSQL (prod)
- **Frontend**: Next.js 14, React 18, TypeScript, Tailwind CSS, Axios (installed but unused)
- **AI Integration**: OpenAI API for content personalization and assessment generation
- **PDF Processing**: PyPDF2, pdfplumber for text extraction and structure analysis

### Core Architecture Pattern
```
PDF Upload → Processing Pipeline → Content Extraction → Personalization → Multi-Modal Generation → Learning Modes
```

**Five Learning Modalities**:
1. **Immersive Text**: Interactive textbook with embedded questions and AI-generated images
2. **Slides & Narration**: Presentation-style content with optional AI narration
3. **Audio Lessons**: Simulated teacher-student conversations with dialogue
4. **Mind Maps**: Hierarchical knowledge visualization with expandable nodes
5. **Assessments**: Quiz system with Bloom's taxonomy levels and progress tracking

## Development Commands

### Project Setup
```bash
# Install all dependencies
npm run install:all

# Development (runs both frontend:3000 and backend:8000)
npm run dev

# Production build
npm run build

# Testing
npm run test
```

### Backend Development
```bash
cd backend
pip3 install -r requirements.txt
python3  main.py                    # Start FastAPI server (port 8000)
uvicorn main:app --reload        # Development server with hot reload
python3  -m pytest                 # Run backend tests
```

### Frontend Development
```bash
cd frontend
npm install
npm run dev                        # Start Next.js dev server (port 3000)
npm run build                      # Production build
npm run test                       # Run React tests (Jest setup ready)
```

### Database
- SQLite database: `learn_your_way.db` (auto-created on first run)
- Models auto-created via SQLAlchemy in `backend/src/core/database.py`
- Tables: Users, Documents, ContentSections, PersonalizedContent, LearningProgress

## Key Integration Points

### 1. PDF Processing Pipeline
**Backend**: Complete service in `backend/src/api/pdf_processing.py`
- Endpoint: `POST /api/pdf/upload` (multipart/form-data)
- Processing: PDF → text extraction → section identification → content analysis
- Storage: File saved to `uploads/` directory, processed JSON saved alongside

**Frontend**: Mock upload in `frontend/src/app/page.tsx:32-38`
- **Missing**: Actual API call to upload endpoint
- **Missing**: Progress tracking during processing
- **Missing**: Error handling and file validation

### 2. Content Management & Personalization
**Backend**: Full CRUD in `backend/src/api/content.py`
- Endpoints: `/api/content/sections`, `/api/content/personalize`
- Personalization engine: `backend/src/services/personalization_service.py`
- Supports: Grade-level adaptation, interest-based content, cultural relevance

**Frontend**: Static content in all learning mode components
- **Missing**: Dynamic content loading from backend
- **Missing**: User preference collection and storage
- **Missing**: Learning mode switching with personalized data

### 3. Assessment System
**Backend**: Quiz generation and grading in `backend/src/api/assessments.py`
- Assessment generator: `backend/src/services/assessment_service.py`
- Support for multiple question types, difficulty levels, feedback generation

**Frontend**: Basic quiz components in `frontend/src/components/learning-modes/AssessmentMode.tsx`
- **Missing**: Real quiz generation from backend
- **Missing**: Quiz submission and grading integration
- **Missing**: Progress tracking and analytics

### 4. Authentication Framework
**Backend**: Structure exists in `backend/src/api/auth.py` but implementation incomplete
- **Missing**: User registration, login/logout functionality
- **Missing**: JWT token management and validation

**Frontend**: No authentication components or state management
- **Missing**: Login/register forms
- **Missing**: Protected route handling
- **Missing**: User session management

## Database Schema Relationships

```
Users (1) → (N) Documents
Users (1) → (N) PersonalizedContent
Users (1) → (N) LearningProgress
Documents (1) → (N) ContentSections
ContentSections (1) → (N) PersonalizedContent
```

**Key Tables**:
- **Users**: grade_level (K-12), interests (JSON), learning_preferences (JSON)
- **Documents**: PDF metadata, processing status, public/private flag
- **ContentSections**: Extracted sections with key concepts and learning objectives
- **PersonalizedContent**: Mode-specific adaptations with personalization metadata
- **LearningProgress**: User progress tracking, quiz scores, time spent

## File Structure Patterns

### Backend Service Layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [THU-MAIC/MAIC-UI](https://github.com/THU-MAIC/MAIC-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
