---
trigger: always_on
description: **Rone Arena API & Web** is a comprehensive, production-grade REST API and web interface for **Mobile Legends: Bang Bang** game data. It is an unofficial, community-maintained project with no affiliation to or endorsement by Moonton; the game name is used descriptively only, and the brand must never incorporate the "MLBB" or "Mobile Legends" marks. It provides developers, analysts, and fans with structured, reliable access to hero information, academy resources, player statistics, and utility to
---

# Rone Arena API & Web - Project Overview

## Project Purpose

**Rone Arena API & Web** is a comprehensive, production-grade REST API and web interface for **Mobile Legends: Bang Bang** game data. It is an unofficial, community-maintained project with no affiliation to or endorsement by Moonton; the game name is used descriptively only, and the brand must never incorporate the "MLBB" or "Mobile Legends" marks. It provides developers, analysts, and fans with structured, reliable access to hero information, academy resources, player statistics, and utility tools.

## Key Features

### 1. **RESTful Public API** (`/api/*`)
- **Hero Data**: Hero listings, rankings, positions, stats, skill combos, counters, relationships
- **Academy Resources**: Roles, equipment, emblems, spells, builds, lane distributions, win rate timelines
- **User Endpoints**: Authentication (with verification codes), profile data, match history, player statistics
- **Utility Tools**: Win rate calculators, IP lookup, hero compatibility analysis
- **Flexible Identifiers**: Support for hero ID or hero name (including slug-like names)
- **OpenAPI/Swagger**: Full OpenAPI 3.0 schema with interactive documentation

### 2. **Web Playground** (`/web/*`)
- **Form-Driven Endpoint Testing**: Interactive forms for all API endpoints
- **Response Viewers**: Readable (Key-Value & Key-As-Header modes) + Raw JSON views
- **Code Snippets**: Auto-generated curl, Python, JavaScript, Go, Node.js, PHP, Java, C# examples
- **Live Execution**: Test endpoints directly from the browser
- **Authentication Modal**: Integrated sign-in flow with JWT caching
- **User Profile Display**: JWT-aware navbar showing profile photo, username, country, role/zone

### 3. **Blog & Tutorials** (`/blog/*`)
- **SEO-Optimized Pages**: Markdown-based guides and release notes
- **Step-by-Step Tutorials**: Getting started, authentication, integration examples

## Architecture

### Backend Stack
- **Framework**: FastAPI (Python 3.12+)
- **Async**: Built with async/await for high concurrency
- **Database**: None (stateless API - all data fetched from upstream game-data services)
- **Upstream Services**: game-data services reached via `RONE_DEV_ACCESS_KEY` / `RONE_DEV_ACCESS_KEY_V2`
- **Templates**: Jinja2 for server-side rendering
- **Static Files**: Tailwind CSS, Alpine.js or vanilla JavaScript

### Frontend Stack
- **Templating**: Jinja2 (server-rendered HTML)
- **Styling**: Tailwind CSS v4
- **JavaScript**: Vanilla JS (no build tools required) with localStorage for session management
- **Interactive Features**: Dynamic forms, code snippet generation, authentication modal

### Deployment
- **Production**: Vercel (runs FastAPI via ASGI)
- **Local Dev**: FastAPI dev server at `http://127.0.0.1:8000`

## Project Structure

```
rone-arena-api/
├── app/
│   ├── api/                    # REST API endpoints
│   │   ├── routers/
│   │   │   ├── user.py         # Authentication & user profiles
│   │   │   ├── heroes.py       # Hero data, stats, analytics
│   │   │   ├── academy.py      # Game guides, builds, resources
│   │   │   ├── addon.py        # Utility endpoints (IP lookup, etc.)
│   │   │   └── root.py         # Root & metadata endpoints
│   │   └── dependencies.py
│   ├── core/
│   │   ├── config.py           # Environment & configuration
│   │   ├── errors.py           # Custom error handling
│   │   ├── exceptions.py       # FastAPI exception handlers
│   │   ├── security.py         # JWT, auth helpers
│   │   ├── enums.py            # Game-related enums
│   │   ├── hero_limits.py      # Hero validation & limits
│   │   └── http.py             # HTTP client for upstream
│   ├── schemas/                # Pydantic models (request/response)
│   ├── services/               # Business logic layer
│   ├── utils/
│   │   ├── client_ip.py
│   │   ├── filters.py
│   ├── web/                    # Web UI (forms, playground)
│   │   ├── routers/
│   │   │   ├── root.py         # Landing page
│   │   │   └── blog.py         # Blog pages
│   │   ├── templates/          # Jinja2 templates
│   │   └── openapi_catalog.py  # Web endpoint metadata
│   └── main.py                 # FastAPI app setup
├── tests/                      # Pytest suite
├── pyproject.toml              # Dependencies & metadata
├── .env.example                # Environment template
└── README.md                   # User documentation
```

## Configuration

### Environment Variables

**Core Settings**:
- `DEBUG`: Set to `true` in dev; when true, API playground uses `http://127.0.0.1:8000/api` locally
- `SECRET_KEY`: Secret for signing JWTs (required)
- `IS_MAINTENANCE`: Set to `true` to restrict the API and show the maintenance page on `/`
- `IS_HIGH_TRAFFIC`: Set to `true` to restrict the API and point callers at the high-volume host

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ridwaanhall/rone-arena-api](https://github.com/ridwaanhall/rone-arena-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
