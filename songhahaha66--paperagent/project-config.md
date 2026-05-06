---
trigger: always_on
description: PaperAgent is an intelligent academic paper writing assistant built with Vue 3 + TypeScript frontend and FastAPI + Python backend. The system uses multiple AI agents to automatically generate academic papers with embedded code execution, data analysis, and visualization.
---

# PaperAgent - AI-Powered Academic Paper Writing Assistant

PaperAgent is an intelligent academic paper writing assistant built with Vue 3 + TypeScript frontend and FastAPI + Python backend. The system uses multiple AI agents to automatically generate academic papers with embedded code execution, data analysis, and visualization.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Quick Start (Complete Bootstrap)
Bootstrap, build, and test the complete application:

1. **Backend Setup**:
   ```bash
   cd backend
   pip install uv
   uv venv
   uv pip install -r pyproject.toml  # takes ~15 seconds. NEVER CANCEL.
   cp .env.example .env
   # Edit .env file - see Database Configuration below
   source .venv/bin/activate  # Linux/macOS
   # For Windows: .venv\Scripts\activate.bat (CMD) or .venv\Scripts\Activate.ps1 (PowerShell)
   ```

2. **Frontend Setup**:
   ```bash
   cd frontend
   npm install  # takes ~14 seconds. NEVER CANCEL.
   ```

3. **Database Configuration** (Critical):
   Edit `backend/.env` with one of these options:
   - **SQLite (Development/Testing)**: `DATABASE_URL=sqlite:///./paperagent.db`
   - **PostgreSQL (Production)**: `DATABASE_URL=postgresql://username:password@localhost:5432/paperagent`

4. **Start Services**:
   ```bash
   # Terminal 1: Backend (takes ~2 seconds to start)
   cd backend && source .venv/bin/activate && uv run main.py
   
   # Terminal 2: Frontend (takes ~1 second to start)
   cd frontend && npm run dev
   ```

### Build Commands
- **Frontend Build**: `npm run build` -- takes ~19 seconds. NEVER CANCEL. Set timeout to 30+ minutes.
- **Frontend Type Check**: `npm run type-check` -- takes ~5 seconds. NEVER CANCEL.
- **Frontend Format**: `npm run format` -- takes ~2 seconds.
- **Backend Import Test**: `python test_imports.py` -- takes ~2 seconds.

## Database Requirements

**CRITICAL**: The application requires a database connection to start. Choose one:

### Option 1: SQLite (Recommended for Development)
- **No setup required** - just set `DATABASE_URL=sqlite:///./paperagent.db` in `.env`
- Database tables are automatically created on first startup
- Perfect for development, testing, and code changes

### Option 2: PostgreSQL (Production Setup)
- **Requires PostgreSQL 12+ installation and running service**
- Create database: `createdb paperagent`
- Set `DATABASE_URL=postgresql://username:password@localhost:5432/paperagent`
- Database tables are automatically created on first startup

**Database Connection Failure**: If backend fails with "connection refused" errors, the database is not running or URL is incorrect.

## Validation

### Manual Testing Scenarios
After making changes, ALWAYS test these scenarios:

1. **Backend Health Check**:
   ```bash
   curl http://localhost:8000/health
   # Expected: {"status":"healthy"}
   ```

2. **API Documentation Access**:
   - Navigate to `http://localhost:8000/docs`
   - Should display FastAPI Swagger UI

3. **Frontend Application**:
   - Navigate to `http://localhost:5173/`
   - Should display PaperAgent login/introduction page
   - Test basic navigation between pages

4. **Backend-Frontend Integration**:
   - Ensure backend (port 8000) and frontend (port 5173) can communicate
   - Check browser console for CORS or API errors

### Pre-Commit Validation
ALWAYS run these commands before committing changes:
```bash
cd frontend && npm run format && npm run type-check
cd backend && python test_imports.py
```

## Common Issues and Solutions

### Backend Won't Start
- **Database Connection Error**: Check DATABASE_URL in `.env`, use SQLite for testing
- **Import Errors**: Run `uv pip install -r pyproject.toml` to reinstall dependencies
- **Port 8000 in use**: Kill existing process with `pkill -f "uvicorn\|main.py"`

### Frontend Won't Start
- **Port 5173 in use**: Kill existing process or use `npm run dev -- --port 5174`
- **Node version**: Requires Node.js 20.19.0+ || 22.12.0+
- **Dependencies missing**: Run `npm install` to reinstall

### API Communication Errors
- **CORS errors**: Backend is configured for wildcard origins - check if backend is running
- **Network errors**: Ensure backend is accessible at `http://localhost:8000`

## Environment Requirements

### Required Software
```bash
# Backend
Python 3.10+
pip (for installing uv)
uv (Python package manager)

# Frontend  
Node.js 20.19.0+ || 22.12.0+
npm (included with Node.js)

# Database (choose one)
SQLite (built into Python)
PostgreSQL 12+ (optional, for production)
```

## Project Structure and Key Locations

### Backend (`backend/`)
- **Entry Point**: `main.py` - FastAPI application with automatic database table creation
- **AI System**: `ai_system/` - Multi-agent AI framework with MainAgent, CodeAgent
- **API Routes**: `routers/` - Authentication, chat, templates, workspace management
- **Database**: `models/models.py` - SQLAlchemy ORM models, `database/database.py` - DB connections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [songhahaha66/PaperAgent](https://github.com/songhahaha66/PaperAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
