---
trigger: always_on
description: This repository contains a sophisticated, multi-layered defense system against AI-powered web scrapers and malicious bots. These instructions help GitHub Copilot coding agent understand the project[...]
---

# GitHub Copilot Instructions for AI Scraping Defense

This repository contains a sophisticated, multi-layered defense system against AI-powered web scrapers and malicious bots. These instructions help GitHub Copilot coding agent understand the project[...]

## Project Overview

**AI Scraping Defense** is a microservice-based security platform that combines:
- Nginx reverse proxy with Lua scripting for first-line defense
- Python microservices for intelligent traffic analysis
- Machine learning models for bot detection
- LLM integration for advanced behavioral analysis
- Active countermeasures (tarpit, honeypots)
- Redis for caching and blocklists
- PostgreSQL for persistence
- Docker/Kubernetes deployment

## Key Technologies

- **Languages**: Python 3.11+, Lua, Rust (some components), Shell scripts
- **Web Frameworks**: FastAPI, Uvicorn, Gunicorn
- **Databases**: Redis, PostgreSQL
- **ML/AI**: scikit-learn, XGBoost, OpenAI, Anthropic, Google GenAI, Cohere, Mistral
- **Infrastructure**: Docker, Kubernetes, Nginx
- **Testing**: pytest, pytest-asyncio
- **Linting/Formatting**: black, isort, flake8
- **CI/CD**: GitHub Actions

## Repository Structure

```
├── .github/              # GitHub Actions workflows and templates
│   ├── workflows/        # CI/CD pipelines (tests, security audits, autofix)
│   ├── ISSUE_TEMPLATE/   # Issue templates
│   └── PULL_REQUEST_TEMPLATE.md
├── src/                  # Core Python microservices
│   ├── admin_ui/         # Admin dashboard service
│   ├── ai_service/       # AI webhook receiver
│   ├── behavioral/       # Behavioral analysis modules
│   ├── bot_control/      # Bot management and rate limiting
│   ├── captcha/          # CAPTCHA verification service
│   ├── cloud_dashboard/  # Cloud monitoring dashboard
│   ├── config_recommender/ # AI-driven configuration recommendations
│   ├── escalation/       # Escalation engine for threat analysis
│   ├── pay_per_crawl/    # Crawler authentication and billing
│   ├── plugins/          # Plugin API for custom rules
│   ├── public_blocklist/ # Community blocklist service
│   ├── rag/              # RAG (Retrieval-Augmented Generation) tools
│   ├── security/         # Security modules (RBAC, audit logging)
│   ├── tarpit/           # Tarpit API for bot resource waste
│   ├── shared/           # Shared utilities and helpers
│   └── util/             # General utilities
├── test/                 # Unit and integration tests (mirrors src/ structure)
├── scripts/              # Setup, deployment, and maintenance scripts
├── nginx/                # Nginx configuration and Lua scripts
├── helm/                 # Kubernetes Helm charts
├── docs/                 # Project documentation
├── AGENTS.md             # Guidelines for automated agents (includes pre-commit, testing)
├── CONTRIBUTING.md       # Contribution guidelines
├── README.md             # Project overview and setup instructions
└── docker-compose.yaml   # Local development environment
```

## Development Workflow

### 1. Environment Setup

Before making changes:
```bash
# Install Python dependencies
pip install -r requirements.txt

# Install pre-commit hooks
pre-commit install
```

### 2. Code Quality and Linting

**Always run pre-commit hooks on modified files:**
```bash
pre-commit run --files <file1> [file2 ...]
```

The project uses:
- **black**: Python code formatting (respects the max line length configured in the repository)
- **isort**: Import sorting
- **flake8**: Python linting (see `.flake8` for configuration including max line length)

Configuration files:
- `.flake8` - Flake8 settings
- `.isort.cfg` - Import sorting configuration
- `.pre-commit-config.yaml` - Pre-commit hook definitions

### 3. Testing

**Run tests before and after changes:**
```bash
# Run all tests
python -m pytest

# Run specific test file
python -m pytest test/path/to/test_file.py

# Run with coverage
python -m pytest --cov=src --cov-report=html
```

Test guidelines:
- Tests are located in `test/` directory, mirroring the `src/` structure
- Use pytest fixtures defined in `test/conftest.py`
- Write unit tests for new functionality
- Integration tests should use Docker Compose when needed
- Mock external services (LLMs, payment gateways, etc.) in tests

### 4. Building and Running Locally

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f [service_name]

# Stop services
docker-compose down
```

### 5. Commit Guidelines

- Use clear, descriptive commit messages in present tense
- Examples: `Add advanced header analysis`, `Fix rate limit daemon crash`, `Update architecture docs`
- Separate unrelated changes into different commits
- Reference issue numbers when applicable: `Fix #123: Resolve memory leak in escalation engine`

## Coding Standards and Best Practices

### Python Code

1. **Follow PEP 8** with these specifics:
   - Line length: Use the max line length configured in the repository's tooling (see `.flake8`, `pyproject.toml` (if present), or formatter-specific configs). Do not assume or hard-code a specif[...]
   - Use type hints for function signatures

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhamenator/ai-scraping-defense](https://github.com/rhamenator/ai-scraping-defense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
