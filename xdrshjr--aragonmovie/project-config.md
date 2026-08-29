---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI短剧自动化生成系统 (AI Drama Automation System) - A multi-agent architecture system that generates complete short drama videos from text scripts. The system provides **dual interfaces**:

1. **CLI Interface**: Direct command-line access for project management and video generation
2. **Web Interface**: FastAPI backend + Next.js frontend for browser-based workflows

Both interfaces share the same core video generation pipeline, which integrates Nano Banana Pro (image generation), Veo3/Sora2 (video generation with runtime selection), and optional LLM services for character consistency judging.

## Core Commands

### Development Setup
```bash
# Create and activate virtual environment
python -m venv venv
.\venv\Scripts\activate  # Windows
source venv/bin/activate  # macOS/Linux

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env to add API keys
```

### Testing
```bash
# Run all tests
python -m pytest

# Run specific test file
python -m pytest tests/test_agents/test_image_video_generation.py

# Run with verbose output and short traceback
python -m pytest tests/test_agents/test_image_video_generation.py -v --tb=short

# Run specific test class
python -m pytest tests/test_agents/test_image_video_generation.py::TestConcurrencyUtilities -v --tb=short

# Run async tests
python -m pytest tests/test_agents/test_script_parser.py -v
```

### Project Initialization
```bash
# Initialize project structure
python init_project.py

# Run example workflow
python examples/complete_workflow_example.py
```

### CLI Interface

**CLI Commands**:
```bash
# Create new project
python cli.py init my_drama

# Generate complete drama video
python cli.py generate projects/my_drama

# Validate project configuration
python cli.py validate projects/my_drama

# List all projects
python cli.py list

# Advanced options
python cli.py generate projects/my_drama --log-level DEBUG
python cli.py generate projects/my_drama --override video.fps=60
python cli.py generate projects/my_drama --resume  # Resume from checkpoint
python cli.py generate projects/my_drama --skip-characters  # Skip character reference generation
```

### Web Interface (Backend + Frontend)

**Backend (FastAPI)**:
```bash
# Run development server
cd backend
python run_dev.py

# Server will start at http://localhost:8000
# API docs: http://localhost:8000/docs
# ReDoc: http://localhost:8000/redoc
```

**Frontend (Next.js)**:
```bash
# Install dependencies
cd frontend
npm install

# Run development server
npm run dev
# Starts at http://localhost:3000

# Build for production
npm run build
npm start

# Type check
npm run type-check
```

### Code Quality
```bash
# Format code
black .

# Lint code
flake8

# Type check
mypy .
```

## System Architecture

### Dual Interface Architecture

The system operates with two distinct entry points that share the same core generation pipeline:

**CLI Path**: `cli.py` → `ProjectManager` → `ProjectRunner` → Agents
**Web Path**: Frontend → `FastAPI` → `WorkflowService` → `ProjectRunner` → Agents

Both paths converge at the agent layer, ensuring consistent video generation logic.

### Backend API Architecture (FastAPI)

**API Router Structure** (`backend/api/router.py`):
- **REST API v1** (`/api/v1/*`):
  - `/llm/*` - LLM services (script polishing, judging)
  - `/images/*` - Image generation endpoints
  - `/videos/*` - Video generation endpoints
  - `/tasks/*` - Async task management
  - `/workflow/*` - Complete workflow orchestration
  - `/projects/*` - Project CRUD operations

- **OpenAI-Compatible API** (`/v1/*`):
  - `/chat/completions` - Chat completion endpoint
  - `/images/generations` - Image generation endpoint
  - `/videos/generations` - Video generation endpoint

**Key Backend Services**:
1. **TaskManager** (`backend/core/task_manager.py`):
   - Async task submission and tracking
   - In-memory task storage with automatic cleanup
   - Status callback system for project synchronization
   - Semaphore-based concurrency control

2. **WorkflowService** (`backend/core/workflow_service.py`):
   - Bridges FastAPI and CLI-based generation logic
   - Creates temporary project structures for web-based workflows
   - Manages asset URL generation and storage
   - Progress callback translation

3. **ProjectManager** (`backend/core/project_manager.py`):
   - Project persistence and metadata management
   - Task-to-project synchronization
   - Status tracking and result aggregation

**Middleware**:
- CORS middleware for frontend communication
- Custom logging middleware
- Exception handlers for consistent error responses

### Frontend Architecture (Next.js)

**Tech Stack**:
- Next.js 14.2 (App Router)
- React 18.3
- TypeScript 5.3
- Tailwind CSS 3.4

**Key Pages**:
- `/` - Home page with project grid
- `/projects/new` - New project creation wizard
- `/projects/[projectId]` - Project detail view
- `/workflow/[projectId]` - Workflow execution page

**Component Structure**:
- Step-based workflow (Step0-Step5)
- Reusable UI components (`components/ui/*`)
- ModeSidebar for workflow type selection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xdrshjr/AragonMovie](https://github.com/xdrshjr/AragonMovie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
