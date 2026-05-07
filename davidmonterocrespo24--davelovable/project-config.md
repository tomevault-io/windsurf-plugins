---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a lovable.dev clone - a web application that allows users to create web projects with AI assistance. The system consists of:

- **Frontend** (`/front`): React + TypeScript + Vite with visual code editor
- **Backend** (`/backend`): FastAPI + SQLite + Microsoft AutoGen for AI agent orchestration

The project supports **physical file storage** for WebContainers integration - generated code is saved both in the database and as physical files for browser-based preview execution.

## Development Commands

### Frontend (in `/front` directory)

```bash
# Install dependencies
npm install

# Run development server (runs on http://localhost:8080)
npm run dev

# Build for production
npm run build

# Lint code
npm run lint

# Preview production build
npm run preview
```

### Backend (in `/backend` directory)

**Initial setup:**
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env and add your OPENAI_API_KEY

# Initialize database
python init_db.py
```

**Running the backend:**
```bash
# Using the run script
python run.py

# Or using uvicorn directly
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Backend runs on http://localhost:8000 with API documentation at http://localhost:8000/docs

## File Storage & Version Control System

The backend uses a **filesystem + Git architecture**:

1. **SQLite Database** - Stores only file metadata (filename, filepath, language, timestamps)
2. **Physical Filesystem** - Stores actual file content at `backend/projects/project_{id}/`
3. **Git Repositories** - Each project is a Git repo for version control

**IMPORTANT:** File content is **NOT** stored in the database. It's stored only in the filesystem and versioned with Git.

**File System Service** ([backend/app/services/filesystem_service.py](backend/app/services/filesystem_service.py)):
- `create_project_structure(project_id, name)` - Creates complete Vite + React project + Git init
- `write_file(project_id, filepath, content)` - Writes file to disk
- `read_file(project_id, filepath)` - Reads file from disk
- `get_all_files(project_id)` - Returns all files for WebContainers bundle

**Git Service** ([backend/app/services/git_service.py](backend/app/services/git_service.py)):
- `init_repository(project_id)` - Initialize Git repo
- `commit_changes(project_id, message, files)` - Commit changes
- `get_commit_history(project_id, limit)` - Get commit log
- `get_file_at_commit(project_id, filepath, commit_hash)` - Get file at specific commit
- `restore_commit(project_id, commit_hash)` - Restore to previous commit

**Project Structure:**
```
backend/projects/project_X/
├── .git/                  # Git repository for version control
├── .gitignore
├── package.json           # Vite + React + TypeScript + Tailwind
├── vite.config.ts
├── tsconfig.json
├── tsconfig.node.json
├── tailwind.config.js
├── postcss.config.js
├── index.html
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── index.css
    └── components/        # AI-generated components
```

**Bundle Endpoint:** `GET /api/v1/projects/{id}/bundle` returns all files (read from filesystem) in format for WebContainers API

**Git Commits:** Every file change creates a Git commit with descriptive message

## WebContainers Integration

The preview uses **WebContainers** (by StackBlitz) to run Node.js directly in the browser:

**Service:** [front/src/services/webcontainer.ts](front/src/services/webcontainer.ts)
- `loadProject(projectId)` - Fetches files, runs npm install, starts dev server
- Converts flat file structure to WebContainer tree format
- Returns server URL for iframe preview

**Preview Component:** [front/src/components/editor/PreviewPanelWithWebContainer.tsx](front/src/components/editor/PreviewPanelWithWebContainer.tsx)
- Initializes WebContainer on mount
- Shows real-time console output (npm install, Vite dev server)
- Displays running app in iframe
- Device preview modes (mobile, tablet, desktop)

**Benefits:**
- ✅ No backend compute for preview (runs in browser)
- ✅ Real Node.js + npm + Vite in browser
- ✅ Hot Module Replacement (HMR) works
- ✅ Infinite scalability (each user = own container)
- ✅ Offline capable after initial load

**Requirements:** Chrome/Edge 89+, requires COOP/COEP headers (configured in vite.config.ts)

See [WEBCONTAINERS_IMPLEMENTATION.md](WEBCONTAINERS_IMPLEMENTATION.md) for full details.

## Architecture

### Backend Multi-Agent System

The backend uses **Microsoft AutoGen** to orchestrate four specialized AI agents that collaborate to generate code:

1. **Architect** - Plans system structure and component architecture
2. **UI Designer** - Designs visual interfaces using Tailwind CSS
3. **Coding Agent** - Generates TypeScript/React code
4. **Code Reviewer** - Reviews code for quality, bugs, and improvements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidmonterocrespo24/DaveLovable](https://github.com/davidmonterocrespo24/DaveLovable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
