---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Quick Start

### Development Setup

```bash
# Environment setup
python -m venv venv
venv\Scripts\activate              # Windows
source venv/bin/activate           # Linux/macOS
pip install -r requirements.txt

# Configuration
# Create .env file and add your OPENAI_API_KEY

# Start development server (port 6217)
python run.py
```

**Access Points:**
- Web Interface: http://localhost:6217
- Default Login: admin@example.com / admin123
- API Documentation: http://localhost:6217/api/docs

### Testing & Code Quality

```bash
# Code formatting and quality
pre-commit install                   # setup hooks
pre-commit run --all-files          # run all checks
autopep8 --in-place --recursive .   # format code
isort .                             # sort imports

# Pre-commit runs these automatically:
# - autopep8 with max line length 127
# - isort with black profile
# - flake8 code quality checks
# - trailing whitespace removal
```

---

## Architecture Overview

AutoModerate is a Flask-based content moderation platform with OpenAI integration and real-time WebSocket updates.

### Core Technology Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Backend** | Flask 2.3.3 + Flask-SocketIO | Web framework with real-time capabilities |
| **Database** | SQLAlchemy (SQLite dev, PostgreSQL prod) | ORM with connection pooling |
| **AI Integration** | OpenAI API (GPT models) | Content analysis and moderation |
| **Authentication** | Flask-Login + API Keys | Session-based web auth + API authentication |
| **Real-time** | WebSocket (Flask-SocketIO) | Live moderation result updates |
| **Caching** | In-memory caches | Rule caching + AI result caching |

### Application Structure

```
AutoModerate/
├── run.py                      # Application entry point (port 6217)
├── config/
│   ├── config.py               # Environment-based configuration
│   └── default_rules.py        # Default moderation rules
├── app/
│   ├── __init__.py             # Flask app factory with database initialization
│   ├── models/                 # SQLAlchemy database models
│   │   ├── user.py             # User authentication and management
│   │   ├── project.py          # Projects with member management
│   │   ├── api_key.py          # API authentication tokens
│   │   ├── api_user.py         # API user tracking
│   │   ├── content.py          # Content submissions for moderation
│   │   ├── moderation_rule.py  # Custom moderation rules
│   │   └── moderation_result.py# Moderation decisions and metadata
│   ├── routes/                 # Blueprint-based routing
│   │   ├── auth.py             # Authentication (login/register/profile)
│   │   ├── dashboard.py        # Web interface for project management
│   │   ├── api.py              # RESTful API for content moderation
│   │   ├── websocket.py        # Real-time WebSocket endpoints
│   │   ├── admin.py            # Admin interface for system management
│   │   └── manual_review.py    # Human review interface
│   ├── services/               # Business logic layer
│   │   ├── moderation_orchestrator.py  # Main workflow coordinator
│   │   ├── database_service.py         # Centralized database operations
│   │   ├── ai/                         # OpenAI integration services
│   │   │   ├── ai_moderator.py         # AI moderation strategies with chunking
│   │   │   ├── openai_client.py        # OpenAI client management
│   │   │   └── result_cache.py         # AI result caching
│   │   └── moderation/                 # Core moderation logic
│   │       ├── rule_processor.py       # Rule evaluation (keyword/regex/AI)
│   │       ├── rule_cache.py           # Rule caching
│   │       └── websocket_notifier.py   # Real-time update handling
│   ├── templates/              # Jinja2 templates for web interface
│   ├── static/                 # CSS, JS assets (modular structure)
│   └── utils/                  # Utility functions
└── docker/                     # Docker deployment configuration
```

---

## Database Architecture

### Core Models & Relationships

| Model | Purpose | Key Features |
|-------|---------|--------------|
| **User** | Authentication & management | UUID primary keys, password hashing, admin roles |
| **Project** | Moderation workspaces | Multi-member support, role-based access (owner/admin/member) |
| **ProjectMember** | Project membership | User-project relationships with roles |
| **ProjectInvitation** | Project invites | Token-based invitation system |
| **APIKey** | API authentication | Auto-generated keys (`am_` prefix), usage tracking |
| **APIUser** | API user tracking | External user ID mapping, usage statistics |
| **Content** | Submitted content | JSON metadata, status tracking, API user association |
| **ModerationRule** | Custom rules | Priority-based, multiple types (keyword/regex/AI prompt) |
| **ModerationResult** | Moderation decisions | Confidence scores, processing metrics, detailed metadata |

### Key Relationships

- User -> Projects (1:N ownership)
- User <-> Projects (N:M membership via ProjectMember)
- Project -> APIKeys, Content, ModerationRules (1:N)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Significant-Gravitas/AutoModerate](https://github.com/Significant-Gravitas/AutoModerate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
