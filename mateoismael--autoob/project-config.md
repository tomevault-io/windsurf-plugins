---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the SEACE Scraper codebase.
---

# CLAUDE.md - AI Assistant Guide for SEACE Scraper

This document provides comprehensive guidance for AI assistants working with the SEACE Scraper codebase.

## Project Overview

**SEACE Scraper** (Automatizador de Búsqueda SEACE) is a full-stack web application that automates the discovery of Peruvian government procurement contracts. It scrapes the SEACE portal (Sistema Electrónico de Contrataciones del Estado) to find technology and goods/services contracts under 8 UIT (Unit of Tax Reference) and provides a time-based color-coded alert system for deadline tracking.

### Key Features
- Automated search for small government contracts (< 8 UIT)
- Color-coded time alerts: 🔴 Red (<24h), 🟡 Yellow (24-72h), 🟢 Green (>72h)
- Real-time web scraping with Playwright
- Modern React UI with responsive design
- RESTful API backend

### Project Status
- **Current State:** Fully functional with Playwright scraper enabled; uses public SEACE search (no login required)
- **Language:** Primarily Spanish documentation and UI
- **Environment:** Development-ready with complete backend setup, not production-deployed
- **Note:** Network connectivity required for scraping; may not work in sandboxed environments

---

## Architecture

### Tech Stack

**Backend (Python)**
- **Framework:** FastAPI 0.121.2 (modern async web framework)
- **Server:** Uvicorn 0.38.0 (ASGI server, port 8002)
- **Scraping:** Playwright 1.56.0 (headless browser automation)
- **Validation:** Pydantic 2.12.4 (type-safe data models)
- **Config:** python-dotenv 1.0.1 (environment variables)
- **Python Version:** 3.9+

**Frontend (TypeScript/React)**
- **Framework:** React 19.1.0 (latest with new features)
- **Language:** TypeScript 5.8.3 (strict mode enabled)
- **Build Tool:** Vite 7.0.4 (fast HMR, port 5173)
- **Styling:** Tailwind CSS 4.1.11 (utility-first CSS)
- **Linting:** ESLint 9.30.1 with TypeScript support
- **Node Version:** 18+

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Browser (localhost:5173)             │
└────────────────────────────┬────────────────────────────────┘
                             │
                    HTTP GET Request
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│              React Frontend (Vite Dev Server)                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  App.tsx - Main Component                            │  │
│  │  - State management (useState hooks)                 │  │
│  │  - API calls to backend                              │  │
│  │  - Table rendering with color coding                 │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────┘
                             │
                    fetch("/api/contrataciones")
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│          FastAPI Backend (Uvicorn on port 8002)              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  main.py - API Endpoints                             │  │
│  │  - CORS middleware (allows localhost:5173)           │  │
│  │  - GET /api/contrataciones                           │  │
│  │  - Currently returns mock data                       │  │
│  └──────────────────┬───────────────────────────────────┘  │
│                     │ (commented out)                       │
│                     ▼                                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  scraper.py - Playwright Web Scraper                 │  │
│  │  - SEACEScraper class                                │  │
│  │  - extraer_contrataciones() method                   │  │
│  │  - calcular_estado_tiempo() for alerts              │  │
│  │  - Headless Chrome automation                        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────┘
                             │
                    Scrapes data from
                             │
                             ▼
                  SEACE Portal (prod6.seace.gob.pe)
```

---

## File Structure

```
autoob/
├── .env.example              # Template for credentials (DO NOT commit .env)
├── .gitignore               # Comprehensive ignore rules (Python, Node, secrets)
├── README.md                # Main documentation (Spanish, 204 lines)
├── QUICKSTART.md            # Quick start guide (Spanish, 101 lines)
├── CLAUDE.md                # This file - AI assistant guide
├── package-lock.json        # Root npm lock file
│
├── backend/                 # Python FastAPI backend
│   ├── main.py              # FastAPI server (118 lines) ⭐ Entry point
│   ├── scraper.py           # Web scraping logic (175 lines) ⭐ Core functionality
│   ├── models.py            # Pydantic models (20 lines)
│   ├── requirements.txt     # Python dependencies
│   ├── .gitignore          # Backend-specific ignores
│   └── venv/               # Python virtual environment (gitignored)
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mateoismael/autoob](https://github.com/mateoismael/autoob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
