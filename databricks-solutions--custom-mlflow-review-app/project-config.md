---
trigger: always_on
description: This is a **production-ready MLflow Review App template** for evaluating AI agents and collecting human feedback on MLflow traces. It provides a complete infrastructure for:
---

# MLflow Review App Development Guide

## Project Overview

This is a **production-ready MLflow Review App template** for evaluating AI agents and collecting human feedback on MLflow traces. It provides a complete infrastructure for:
- **Trace Evaluation**: Review and rate AI agent interactions
- **Custom Schemas**: Define evaluation criteria (ratings, categories, text feedback)
- **SME Workflows**: Assign subject matter experts to review sessions
- **Analytics**: Analyze feedback patterns and inter-rater agreement
- **MLflow Integration**: Direct connection to traces and experiments

## Quick Start for Customization

### 1. Initial Setup
```bash
./setup.sh                    # Interactive environment setup
./watch.sh                    # Start dev servers (frontend:5173, backend:8000)
```

### 2. Customize for Your Use Case

#### Option A: Automated Customization (Recommended)
Use the `/review-app` command in Claude Code for AI-guided setup:
```
/review-app [your experiment description]
```
This will:
- Analyze your MLflow experiment
- Suggest optimal labeling schemas
- Create tailored evaluation criteria
- Set up labeling sessions with appropriate filters

#### Option B: Manual Customization
1. **Update experiment in .env.local**:
   ```bash
   MLFLOW_EXPERIMENT_ID='your_experiment_id'
   SME_THANK_YOU_MESSAGE="Custom thank you message"
   ```

2. **Create custom labeling schemas**:
   ```bash
   ./mlflow-cli run create_labeling_schemas --help
   ```

3. **Set up labeling sessions**:
   ```bash
   ./mlflow-cli run create_labeling_session --help
   ```

### 3. Key Customization Points

#### Frontend Customization
- **`client/src/pages/LabelingPage.tsx`**: Main SME interface
- **`client/src/components/SMELabelingInterface.tsx`**: Core labeling UI
- **`client/src/components/session-renderer/`**: Custom trace renderers
- **`client/src/pages/DeveloperDashboard.tsx`**: Admin interface

#### Backend Customization
- **`server/routers/review/`**: Review app API endpoints
- **`server/routers/mlflow/`**: MLflow proxy endpoints
- **`server/utils/sme_*.py`**: SME analysis utilities
- **`server/models/`**: Data models and types

#### Custom Trace Renderers
Create specialized views for your traces:
```typescript
// client/src/components/session-renderer/renderers/CustomRenderer.tsx
export const CustomRenderer: ItemRenderer = {
  name: 'Custom View',
  description: 'Specialized view for your agent type',
  render: (item) => <YourCustomComponent item={item} />
};
```

## Tech Stack

**Backend:**
- Python with `uv` for package management
- FastAPI for API framework
- Databricks SDK for workspace integration
- MLflow SDK for trace and experiment management
- OpenAPI automatic client generation

**Frontend:**
- TypeScript with React
- Vite for fast development and hot reloading
- shadcn/ui components with Tailwind CSS
- React Query for API state management
- Bun for package management

## Development Workflow

### Package Management
- Use `uv add/remove` for Python dependencies, not manual edits to pyproject.toml
- Use `bun add/remove` for frontend dependencies, not manual package.json edits
- Always check if dependencies exist in the project before adding new ones

### Development Commands
- `./setup.sh` - Interactive environment setup and dependency installation
- `./watch.sh` - Start development servers with hot reloading (frontend:5173, backend:8000)
- `./fix.sh` - Format code (ruff for Python, prettier for TypeScript)
- `./deploy.sh` - Deploy to Databricks Apps

### 🚨 Development Server (CRITICAL) 🚨

**ALWAYS use the watch script - NEVER run servers manually:**

```bash
# Start development servers (REQUIRED COMMAND)
nohup ./watch.sh > /tmp/databricks-app-watch.log 2>&1 &
```

**Why `./watch.sh` is required:**
- Configures environment variables properly
- Starts both frontend and backend correctly
- Generates TypeScript client automatically
- Handles authentication setup
- Provides proper logging and error handling

**Server Details:**
- **Frontend**: http://localhost:5173 (React + Vite)
- **Backend**: http://localhost:8000 (FastAPI)
- **API docs**: http://localhost:8000/docs
- **Hot reloading**: Both frontend and backend
- **Logs**: `/tmp/databricks-app-watch.log`

**Management Commands:**
- **Check logs**: `tail -f /tmp/databricks-app-watch.log`
- **Check status**: `ps aux | grep databricks-app` or check PID file
- **Stop servers**: `pkill -f "watch.sh"` or `kill $(cat /tmp/databricks-app-watch.pid)`

### 🚨 PYTHON EXECUTION RULE 🚨

**NEVER run `python` directly - ALWAYS use `uv run`:**

```bash
# ✅ CORRECT - Always use uv run
uv run python script.py
uv run uvicorn server.app:app
uv run scripts/make_fastapi_client.py

# ❌ WRONG - Never use python directly
python script.py
uvicorn server.app:app
python scripts/make_fastapi_client.py
```

### 🚨 DATABRICKS CLI EXECUTION RULE 🚨

**NEVER run `databricks` CLI directly - ALWAYS prefix with environment setup:**

```bash
# ✅ CORRECT - Always source .env.local and export variables first
source .env.local && export DATABRICKS_HOST && export DATABRICKS_CLIENT_ID && export DATABRICKS_CLIENT_SECRET && export DATABRICKS_WORKSPACE_ID && databricks current-user me


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databricks-solutions/custom-mlflow-review-app](https://github.com/databricks-solutions/custom-mlflow-review-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
