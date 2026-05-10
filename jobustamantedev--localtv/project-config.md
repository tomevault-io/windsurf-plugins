---
trigger: always_on
description: localTv is a web streaming platform for live TV content, built for educational purposes. It features a React frontend with Vite, a FastAPI backend, and SQLite database. This guide helps developers understand the project architecture and how to contribute.
---

# localTv Development Guide

## Project Overview

localTv is a web streaming platform for live TV content, built for educational purposes. It features a React frontend with Vite, a FastAPI backend, and SQLite database. This guide helps developers understand the project architecture and how to contribute.

**Status**: Phase 5 Complete - Integration, Testing, and Documentation Finalized

## Stack Tecnológico Final

### Frontend
- **React 19.2.5** - Modern UI library with hooks
- **Vite 8.0.10** - Lightning-fast build tool and dev server
- **React Router DOM 7.14.2** - Client-side routing
- **Clappr** - HLS/DASH video player (optional integration)
- **CSS3** - Styling with responsive design

### Backend
- **FastAPI 0.111.0** - Async web framework
- **Uvicorn 0.30.0** - ASGI application server
- **SQLAlchemy 2.0.30** - ORM for database
- **SQLite** - Lightweight relational database
- **Pydantic 2.7.0** - Data validation using Python type hints

### DevOps & Deployment
- **Docker** - Container images for backend and frontend
- **Docker Compose** - Orchestration of services
- **Python 3.9+** - Backend runtime
- **Node.js 18+** - Frontend runtime

## Architecture Overview

```
Client Browser (http://localhost:5173)
           ↓
       React App
    (Home + Admin)
           ↓
    HTTP/JSON API
    (CORS enabled)
           ↓
    FastAPI Backend
  (http://localhost:8000)
           ↓
       SQLite DB
   (localTv.db)
```

### Key Decisions

1. **React + Vite**: Fast development experience, modern tooling
2. **FastAPI**: Type-safe, auto-generated API docs (Swagger UI)
3. **SQLite**: Simple setup, no external dependencies
4. **Separate frontend/backend**: Clear separation of concerns, easy to deploy

## Project Structure

```
localTv/
├── backend/
│   ├── app/
│   │   ├── models/              # SQLAlchemy models
│   │   │   ├── __init__.py
│   │   │   ├── channel.py       # Channel model
│   │   │   └── category.py      # Category model
│   │   ├── schemas/             # Pydantic schemas
│   │   │   ├── __init__.py
│   │   │   ├── channel.py
│   │   │   └── category.py
│   │   ├── routers/             # API endpoints
│   │   │   ├── __init__.py
│   │   │   ├── channels.py      # /api/channels endpoints
│   │   │   └── categories.py    # /api/categories endpoints
│   │   ├── crud/                # Database operations
│   │   │   ├── __init__.py
│   │   │   ├── channels.py
│   │   │   └── categories.py
│   │   ├── __init__.py
│   │   ├── auth.py              # API key validation
│   │   ├── config.py            # Environment config
│   │   └── database.py          # DB session setup
│   ├── scripts/
│   │   ├── __init__.py
│   │   └── seed.py              # DB initialization/seeding
│   ├── main.py                  # FastAPI app initialization
│   ├── requirements.txt
│   ├── .env                     # Local env vars (not in git)
│   ├── .env.example             # Template for .env
│   ├── venv/                    # Virtual environment
│   ├── Dockerfile
│   └── .dockerignore
│
├── frontend/
│   ├── src/
│   │   ├── components/          # Reusable React components
│   │   │   ├── Navbar/
│   │   │   ├── ChannelCard/
│   │   │   ├── CategoryFilter/
│   │   │   ├── ProtectedRoute/
│   │   │   └── ...
│   │   ├── pages/               # Page components
│   │   │   ├── Home.jsx
│   │   │   └── Admin.jsx
│   │   ├── context/             # React Context (global state)
│   │   │   └── ChannelContext.jsx
│   │   ├── hooks/               # Custom React hooks
│   │   │   └── useChannels.js
│   │   ├── services/            # API client
│   │   │   └── api.js
│   │   ├── utils/               # Helper functions
│   │   ├── App.jsx              # Root component
│   │   ├── main.jsx             # Entry point
│   │   └── index.css            # Global styles
│   ├── public/                  # Static assets
│   ├── package.json
│   ├── vite.config.js           # Vite configuration
│   ├── .env                     # Local env vars (not in git)
│   ├── .env.example             # Template for .env
│   ├── node_modules/            # Dependencies
│   ├── Dockerfile
│   └── .dockerignore
│
├── scripts/
│   └── start.sh                 # Development startup script
│
├── docs/                        # Documentation
├── specs/                       # Project specifications
│
├── .gitignore                   # Git ignore rules
├── docker-compose.yml           # Docker service orchestration
├── Dockerfile.backend           # Backend container image
├── Dockerfile.frontend          # Frontend container image
├── README.md                    # User documentation
├── CLAUDE.md                    # This file
└── .git/                        # Git repository
```

## Backend Development

### Adding a New Endpoint

1. **Create a model** in `app/models/`:
```python
from sqlalchemy import Column, String, Integer
from app.database import Base

class MyModel(Base):
    __tablename__ = "my_models"
    
    id = Column(Integer, primary_key=True)
    name = Column(String, index=True)
```

2. **Create a schema** in `app/schemas/`:
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jobustamantedev/localTv](https://github.com/jobustamantedev/localTv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
