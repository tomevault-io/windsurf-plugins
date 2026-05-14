---
trigger: always_on
description: **AwesomeIndex** is a search engine for individual projects within GitHub's "Awesome" repositories. Unlike existing solutions that only search repository names and descriptions, AwesomeIndex indexes and searches the actual project entries within these curated lists.
---

# CLAUDE.md

## Project Overview

**AwesomeIndex** is a search engine for individual projects within GitHub's "Awesome" repositories. Unlike existing solutions that only search repository names and descriptions, AwesomeIndex indexes and searches the actual project entries within these curated lists.

## Problem Statement

GitHub's "Awesome" repositories (like `awesome-python`, `awesome-javascript`) contain thousands of curated projects, but there's no way to search for specific projects across all these lists. Developers must manually browse through massive markdown files to find relevant tools and libraries.

## Solution

A web platform that:
- Discovers and monitors Awesome repositories via GitHub API
- Parses markdown content to extract individual project entries
- Indexes projects in MeiliSearch for instant, typo-tolerant search
- Provides a fast, SEO-friendly web interface for discovery

## Technology Stack

### Backend
- **Python 3.12+** - Main language
- **FastAPI** - Web framework
- **SQLModel** - Database ORM with type safety
- **SQLite** - Database (with WAL mode for performance)
- **MeiliSearch** - Search engine with meilisearch-python-sdk (async client)
- **httpx** - HTTP client for API calls
- **uv** - Package management
- **ruff** - Linting and formatting
- **ty** - Type checking (Astral's type checker)

### Frontend
- **HTMX** - Dynamic interactions without JavaScript frameworks
- **Jinja2** - Server-side templating
- **CSS** - Custom responsive styling

### External APIs
- **GitHub API** - Repository discovery and metadata
- **MeiliSearch** - Search indexing and queries

## Project Structure

```
awesomeindex/
├── app/
│   ├── main.py              # FastAPI application entry
│   ├── config.py            # Settings and configuration (.env support)
│   ├── database.py          # SQLite connection setup
│   ├── models/              # SQLModel definitions
│   │   ├── __init__.py      # Model exports
│   │   ├── project.py       # Project model with CRUD schemas
│   │   └── repository.py    # Repository model with CRUD schemas
│   ├── internal/            # Internal business logic (Go-style)
│   │   ├── __init__.py      # Internal module init
│   │   ├── github.py        # GitHub API client with authentication
│   │   ├── parser.py        # Markdown parsing service (awesome list extraction)
│   │   ├── search.py        # MeiliSearch integration
│   │   └── sync.py          # Data synchronization orchestration
│   ├── routers/             # FastAPI route handlers
│   │   ├── __init__.py      # Router setup
│   │   ├── search.py        # Search API endpoints
│   │   └── projects.py      # Project API endpoints
│   ├── templates/           # Jinja2 HTML templates
│   │   ├── base.html        # Base layout with HTMX
│   │   ├── index.html       # Homepage with search interface
│   │   ├── results.html     # Unified results template
│   │   ├── results_more.html # Infinite scroll continuation
│   │   └── _project_item.html # Reusable project display partial
│   └── static/              # Self-hosted assets
│       ├── htmx.min.js      # HTMX library
│       └── styles.css       # Extracted CSS styles
├── .env                     # Environment configuration (GitHub token, DB path)
├── awesomeindex.db         # SQLite database file
├── awesome-repositories-backup.json # Repository data backup
├── pyproject.toml          # uv project configuration
├── uv.lock                 # Dependency lock file
├── README.md               # Project documentation
└── CLAUDE.md              # This file
```

## Key Features

### Core Functionality
- **Repository Discovery**: Automatically find and validate Awesome repositories
- **Content Parsing**: Extract project entries from various markdown formats using regex patterns
- **Full-Text Search**: MeiliSearch-powered search with typo tolerance and instant results
- **Search Interface**: Real-time search with HTMX (300ms delay) and category/repository filtering
- **REST API**: Complete API for search, project browsing, and metadata retrieval
- **Data Enrichment**: Fetch GitHub metadata (stars, language, last update) - planned

### Technical Features
- **Type Safety**: Astral's ty type checker
- **SEO Friendly**: Server-side rendering with HTMX enhancement
- **Responsive Design**: Mobile-first interface

## Data Models

### Repository
- Awesome repository metadata (name, stars, description)
- Sync status and validation flags
- Activity tracking

### Project
- Individual project entries from Awesome lists
- Enriched with GitHub metadata
- Categorization and tagging
- Quality scoring metrics

## API Endpoints

### Search API (`/api/search/`)

-
`GET /api/search/?q={query}&category={category}&repository={repository}&language={language}&min_stars={min_stars}&topics={topics}&sort={sort}&limit={limit}&offset={offset}` -
Search projects with filtering and sorting

### Project API (`/api/projects/`)

- `GET /api/projects/` - List all projects with optional filtering
- `GET /api/projects/{id}` - Get specific project by ID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MorrisonWill/awesomeindex](https://github.com/MorrisonWill/awesomeindex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
