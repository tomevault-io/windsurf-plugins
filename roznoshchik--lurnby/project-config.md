---
trigger: always_on
description: Lurnby is a personal knowledge practice tool built with Flask that helps users read and remember more through active recall and spaced repetition. It supports web articles and epubs with highlighting, categorization, and review features.
---

# Project Overview

Lurnby is a personal knowledge practice tool built with Flask that helps users read and remember more through active recall and spaced repetition. It supports web articles and epubs with highlighting, categorization, and review features.

**Tech Stack:**
- Backend: Flask, SQLAlchemy, PostgreSQL, Redis
- Frontend: Preact + Vite (active migration from Jinja2 templates), JavaScript, CSS
- Background Tasks: RQ (Redis Queue)
- External Services: AWS S3 (epub images), Google OAuth, SendGrid (email)
- Deployment: Heroku (via git push using Dockerfile)
- Testing: pytest (backend), Vitest (frontend)

## Development Setup

### Initial Setup
```bash
# Create and activate virtual environment
python3 -m venv venv
. venv/bin/activate

# Install Python dependencies
pip install -r requirements.txt

# Install Node.js dependencies for Preact frontend
cd client && npm install && cd ..

# Setup database
flask db upgrade

# Create .env file from .env.example
cp .env.example .env
# Edit .env with required credentials
```

### Running the Application

```bash
just serve        # Dev mode: Redis → RQ → Vite → Flask
just serve-prod   # Prod mode: builds assets first
```

### Testing
```bash
just test         # Run all tests (Python + frontend)
just test-python  # Run only Python tests
just test-client  # Run only frontend tests

# Additional options
pytest --cov=app                    # Python tests with coverage
pytest tests/api/test_articles.py   # Specific test file
cd client && npm t                  # Frontend tests with UI
```

### Code Quality
```bash
just format         # Format all code (Python + client)
just format-python  # Format only Python files
just format-client  # Format only client files

just lint           # Lint all code (Python + client)
just lint-python    # Lint only Python files
just lint-client    # Lint only client files

# Pre-commit hooks
pre-commit run --all-files  # Run manually
pre-commit install          # Install hooks
```

**Tools used:**
- Format: Black (Python), Biome (JavaScript/JSX)
- Lint: Flake8 (Python), Biome (client)

**Pre-commit hooks include:**
- Flake8 (serious errors only: syntax errors, undefined names)
- Black (manual stage, for checking only)
- Pytest (Python tests)
- Vitest (frontend tests)
- Trailing whitespace, end-of-file-fixer, check-yaml, etc.

## Architecture

### Application Structure
#### Legacy app being retired
A legacy app that uses jinja templates, wtforms, and inline js + scss is being refactored to a preact frontend using an api as the backend.

The app uses Flask's application factory pattern with blueprints:

- **`learnbetter.py`**: Entry point that creates the Flask app and registers shell context
- **`config.py`**: Configuration management using environment variables
- **`app/__init__.py`**: Application factory (`create_app()`) and extension initialization
- **`app/models/`**: SQLAlchemy models split into separate files (user.py, article.py, highlight.py, event.py, tag.py, etc.)
- **`app/cli.py`**: Custom Flask CLI commands (serve, build, test)

### Blueprints

1. **`app/auth`**: Authentication routes (login, register, password reset)
2. **`app/main`**: Core user-facing routes for reading, highlighting, and reviewing content
3. **`app/content`**: Content management routes
4. **`app/settings`**: User settings and preferences
5. **`app/api`**: RESTful API endpoints (articles, highlights, tags, users, auth)
6. **`app/client`**: Preact frontend routes (serves client-side rendered pages)
7. **`app/dotcom`**: Marketing/landing page routes
8. **`app/errors`**: Error handlers
9. **`app/assets_blueprint`**: Asset resolution for dev/prod environments

### Key Components

**Background Tasks (`app/tasks.py`):**
- Uses RQ (Redis Queue) for async processing
- Handles epub conversion, PDF imports, exports, and email generation
- Tasks update progress via Redis for UI feedback

**Content Processing:**
- **ReadabiliPy**: Mozilla Readability.js wrapper for extracting clean article content from web pages (requires node)
- **EbookLib + PyMuPDF**: EPUB and PDF processing
- **BeautifulSoup + lxml**: HTML parsing and manipulation
- Web article text extraction in `app/helpers/pulltext.py`
- EPUB handling in `app/helpers/ebooks.py`
- PDF handling in `app/helpers/pdf.py`

**Review System (`app/helpers/review.py`):**
- Implements spaced repetition for highlights
- `order_highlights()` function determines review order based on intervals

**Export System (`app/export.py`, `app/helpers/export_helpers.py`):**
- Exports user data (highlights, articles) to various formats
- ZIP file generation for bulk exports

### Data Models

Core relationships:
- **User** → Articles (one-to-many)
- **User** → Highlights (one-to-many)
- **User** → Topics (one-to-many, for categorization)
- **User** → Tags (one-to-many)
- **Article** → Highlights (one-to-many)
- **Highlight** ↔ Topics (many-to-many via `highlights_topics`)
- **Article/Highlight** ↔ Tags (many-to-many via `tags_articles`, `tags_highlights`)

Important model features:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Roznoshchik/Lurnby](https://github.com/Roznoshchik/Lurnby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
