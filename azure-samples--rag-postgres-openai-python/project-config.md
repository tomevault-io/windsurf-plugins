---
trigger: always_on
description: **ALWAYS FOLLOW THESE INSTRUCTIONS FIRST**. Only search for additional information or use bash commands when the instructions below are incomplete or found to be in error.
---

# RAG on PostgreSQL - Development Instructions

**ALWAYS FOLLOW THESE INSTRUCTIONS FIRST**. Only search for additional information or use bash commands when the instructions below are incomplete or found to be in error.

## Overview

RAG on PostgreSQL is a Python FastAPI backend with React TypeScript frontend that provides a web-based chat application using OpenAI models to answer questions about data in a PostgreSQL database with pgvector extension. The application is designed for Azure deployment via Azure Developer CLI (azd).

## Required Tools and Dependencies

Install the following tools before beginning development:

- **Python 3.10+** (3.12 recommended)
- **Node.js 18+** for frontend development
- **PostgreSQL 14+** with pgvector extension
- **Azure Developer CLI (azd)** for deployment
- **Docker Desktop** for dev containers (optional)
- **Git** for version control

## Development Environment Setup

### Bootstrap the Development Environment

Run these commands in sequence. NEVER CANCEL any long-running commands:

1. **Install Python dependencies** (takes ~90 seconds):
   ```bash
   python3 -m pip install -r requirements-dev.txt
   ```

2. **Install backend package in editable mode** (takes ~5 seconds):
   ```bash
   python3 -m pip install -e src/backend
   ```

3. **Install PostgreSQL and pgvector extension**:
   ```bash
   # Ubuntu/Debian:
   sudo apt update && sudo apt install -y postgresql-16-pgvector

   # Start PostgreSQL and set password
   sudo service postgresql start
   sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'postgres'"
   ```

4. **Configure environment file**:
   ```bash
   cp .env.sample .env
   ```
   Edit `.env` to set `POSTGRES_USERNAME=postgres` and `POSTGRES_PASSWORD=postgres`.

5. **Set up database and seed data** (takes ~2 seconds each):
   ```bash
   python ./src/backend/fastapi_app/setup_postgres_database.py
   python ./src/backend/fastapi_app/setup_postgres_seeddata.py
   ```

6. **Install frontend dependencies** (takes ~22 seconds):
   ```bash
   cd src/frontend
   npm install
   cd ../../
   ```

7. **Build frontend** (takes ~12 seconds):
   ```bash
   cd src/frontend
   npm run build
   cd ../../
   ```

8. **Install pre-commit hooks**:
   ```bash
   pre-commit install
   ```

## Running the Application

### Backend Server
```bash
python -m uvicorn fastapi_app:create_app --factory --reload
```
Serves at `http://localhost:8000` with built frontend included.

### Frontend Development Server (with hot reloading)
```bash
cd src/frontend
npm run dev
```
Serves at `http://localhost:5173/` with hot reloading for development.

### Both via VS Code
Use "Frontend & Backend" configuration in the VS Code Run & Debug menu.

## Code Quality and Testing

### Linting and Formatting (ALWAYS run before committing)
```bash
ruff check .          # Lint code (takes <1 second)
ruff format .          # Format code (takes <1 second)
ty check . --python-version 3.12  # Type check
```

### Testing (NEVER CANCEL - full test suite takes ~25 seconds)
```bash
pytest -s -vv --cov --cov-fail-under=85
```

**CRITICAL**: Some tests may fail with database connection issues if using different PostgreSQL credentials. This is expected in fresh environments and does not indicate broken functionality.

### End-to-End Testing with Playwright (NEVER CANCEL - takes 2+ minutes)
```bash
playwright install chromium --with-deps
pytest tests/e2e.py --tracing=retain-on-failure
```

## Build Times and Timeout Requirements

**CRITICAL TIMING INFORMATION** - Set these timeout values and NEVER CANCEL:

- **Dependencies install**: 90 seconds (use 180+ second timeout)
- **Frontend npm install**: 22 seconds (use 60+ second timeout)
- **Frontend build**: 12 seconds (use 30+ second timeout)
- **ty type checking**: use 90+ second timeout
- **Full test suite**: 25 seconds (use 60+ second timeout)
- **Playwright E2E tests**: 2+ minutes (use 300+ second timeout)

## Manual Validation After Changes

**ALWAYS perform these validation steps after making code changes:**

1. **Lint and format code**:
   ```bash
   ruff check . && ruff format .
   ```

2. **Type check (if Python changes)**:
   ```bash
   ty check . --python-version 3.12
   ```

3. **Run relevant tests**:
   ```bash
   pytest tests/test_<relevant_module>.py -v
   ```

4. **Test application end-to-end**:
   ```bash
   # Start server
   python -m uvicorn fastapi_app:create_app --factory --reload
   ```
   Then in another terminal:
   ```bash
   # Test API endpoints
   curl http://localhost:8000/items/1
   # Should return JSON with item data

   # Test frontend
   curl http://localhost:8000/ | head -n 5
   # Should return HTML with "RAG on PostgreSQL" title
   ```

5. **Test frontend build**:
   ```bash
   cd src/frontend && npm run build
   ```

6. **Functional testing scenarios**:
   - Open `http://localhost:8000/` in browser
   - Verify the "Product chat" interface loads with example questions
   - Click an example question (will show Azure auth error in local dev - this is expected)
   - Verify the frontend UI is responsive and properly styled

## Key Project Structure

### Backend (`src/backend/fastapi_app/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/rag-postgres-openai-python](https://github.com/Azure-Samples/rag-postgres-openai-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
