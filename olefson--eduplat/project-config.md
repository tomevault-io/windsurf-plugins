---
trigger: always_on
description: This file provides context and guidelines for AI assistance in the EduPlat project.
---

# EduPlat - Cursor Rules

This file provides context and guidelines for AI assistance in the EduPlat project.

## Project Overview

**EduPlat** is an AI-enabled question-answering and learning support platform for VistaLab at the University of South Florida. The platform provides intelligent tools to enhance student learning through AI-driven question rephrasing, hint generation, supervised explanations, and reasoning flow analysis.

### Key Context
- **Organization**: VISTA Lab @ USF
- **Focus Areas**: AI in Education, Educational Data Mining, Trustworthy AI, Learning Analytics
- **Mission**: Empowering the future of education through intelligent, trustworthy AI systems

## Project Structure

```
EduPlat/
├── front_end/          # React + Vite frontend
│   ├── src/
│   │   ├── components/ # React components
│   │   ├── pages/      # Page components
│   │   ├── services/   # API clients
│   │   └── App.jsx     # Main app
│   └── .benv.example   # Environment variables template
├── back_end/           # FastAPI backend
│   ├── app/
│   │   ├── api/        # API routes
│   │   ├── models/     # Database models (SQLAlchemy)
│   │   ├── services/   # Business logic
│   │   ├── utils/      # Utilities
│   │   ├── scripts/    # Utility scripts
│   │   ├── config.py   # Configuration
│   │   └── main.py     # FastAPI app
│   └── .benv.example   # Environment variables template
├── data/               # Data storage (gitignored)
│   ├── uploads/        # Uploaded files
│   └── edplat.db       # SQLite database
└── docs/               # Documentation
```

## Technology Stack

### Backend
- **Framework**: FastAPI
- **Language**: Python 3.10+
- **Database**: SQLite (via SQLAlchemy)
- **AI Services**: OpenAI GPT-4, Tavily API
- **File Storage**: Local filesystem + OpenAI file uploads
- **Environment**: Python virtual environment (`venv`)

### Frontend
- **Framework**: React 18+
- **Build Tool**: Vite
- **HTTP Client**: Axios
- **Styling**: CSS (consider Tailwind CSS for future)

## Code Style & Conventions

### Python (Backend)
- Follow PEP 8 style guide
- Use type hints where appropriate
- Use descriptive variable and function names
- Keep functions focused and single-purpose
- Use docstrings for modules, classes, and functions
- Prefer f-strings for string formatting
- Use pathlib.Path for file paths
- Import order: standard library, third-party, local imports

### JavaScript/React (Frontend)
- Use functional components with hooks
- Use camelCase for variables and functions
- Use PascalCase for components
- Prefer const/let over var
- Use arrow functions for callbacks
- Keep components small and focused
- Extract reusable logic into custom hooks
- Use async/await for API calls

### File Naming
- **Python**: `snake_case.py` (e.g., `question_processor.py`)
- **React Components**: `PascalCase.jsx` (e.g., `SimpleQA.jsx`)
- **CSS**: `PascalCase.css` matching component name
- **Config Files**: lowercase (e.g., `config.py`, `main.py`)

## Architecture Patterns

### Backend Architecture
- **API Routes** (`app/api/`): Handle HTTP requests/responses, validation
- **Services** (`app/services/`): Business logic, AI integration, data processing
- **Models** (`app/models/`): Database models and schemas
- **Utils** (`app/utils/`): Helper functions, utilities
- **Config** (`app/config.py`): Centralized configuration management

### Frontend Architecture
- **Components** (`src/components/`): Reusable UI components
- **Pages** (`src/pages/`): Page-level components
- **Services** (`src/services/`): API clients, external service integrations
- **App.jsx**: Main application component with routing

### Data Flow
1. Frontend makes API request → Backend API route
2. API route validates → Calls service layer
3. Service processes → Interacts with database/models
4. Service returns data → API route formats response
5. Frontend receives → Updates UI

## Environment Variables

### Important: Use `.benv` not `.env`
- We use `.benv` files so they're visible and editable before going live
- Never commit `.benv` files (they're in `.gitignore`)
- Always commit `.benv.example` files as templates
- Load `.benv` files using `python-dotenv` in backend
- Use `VITE_` prefix for frontend environment variables (Vite requirement)

### Backend Environment Variables
- `OPENAI_API_KEY`: OpenAI API key (required)
- `TAVILY_API_KEY`: Tavily API key (optional)
- `UPLOAD_DIR`: Path to upload directory
- `DATABASE_PATH`: Path to SQLite database
- `BACKEND_PORT`: Server port (default: 8000)
- `CORS_ORIGINS`: Comma-separated list of allowed origins

### Frontend Environment Variables
- `VITE_API_BASE_URL`: Backend API URL
- Use `import.meta.env.VITE_*` to access in code

## Database Patterns

### SQLAlchemy Models
- All models in `app/models/database.py`
- Use SQLAlchemy declarative base
- Define relationships explicitly
- Use foreign keys for relationships
- Store file paths and metadata appropriately

### Database Initialization
- Use `app/scripts/init_db.py` to initialize database
- Always create tables using `Base.metadata.create_all()`
- Ensure data directories exist before database operations

### Common Patterns
```python
# Get database session
from app.models.database import get_db, SessionLocal

# In route handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/olefson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
