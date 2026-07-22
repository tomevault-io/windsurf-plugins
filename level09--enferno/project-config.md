---
trigger: always_on
description: This file provides comprehensive architectural patterns and coding standards for AI agents working with the Enferno framework.
---

# AGENTS.md

This file provides comprehensive architectural patterns and coding standards for AI agents working with the Enferno framework.

## Framework Overview

**Enferno** is a Flask-based web framework optimized for rapid development with modern tooling:

- **Backend**: Flask 3.x with Blueprint organization
- **Frontend**: Vue 3 + Vuetify (no build step required)
- **Database**: SQLAlchemy 2.x with modern statement-based queries
- **Auth**: Flask-Security-Too with 2FA, WebAuthn, OAuth support
- **Tasks**: Celery + Redis for background processing
- **Package Manager**: uv for fast dependency management

## Project Structure

```
enferno/
├── enferno/                # Main application package
│   ├── app.py             # Application factory (create_app)
│   ├── settings.py        # Environment-based configuration
│   ├── extensions.py      # Flask extensions initialization
│   ├── commands.py        # Custom Flask CLI commands
│   ├── public/            # Public routes (no auth)
│   │   ├── views.py
│   │   ├── models.py
│   │   └── templates/
│   ├── user/              # Authentication & user management
│   │   ├── views.py
│   │   ├── models.py
│   │   ├── forms.py
│   │   └── templates/
│   ├── portal/            # Protected dashboard/admin
│   │   ├── views.py
│   │   └── templates/
│   ├── services/          # Business logic layer
│   ├── tasks/             # Celery task definitions
│   ├── utils/             # Utility functions
│   ├── static/            # CSS, JS, images
│   │   ├── css/
│   │   ├── js/
│   │   └── img/
│   └── templates/         # Global Jinja2 templates
├── docs/                  # Documentation
├── nginx/                 # Nginx configuration
├── instance/              # Instance-specific files (gitignored)
├── pyproject.toml         # Dependencies and project metadata
├── uv.lock               # Lock file for reproducible installs
├── .env                   # Environment variables (gitignored)
├── .env-sample            # Environment template
├── setup.sh              # Setup script
├── run.py                # Application entry point
├── Dockerfile            # Docker configuration
└── docker-compose.yml    # Docker Compose orchestration
```

## Development Commands

### Setup & Installation
```bash
./setup.sh                    # Create virtual environment, install dependencies, generate .env
uv sync --extra dev           # Install dependencies with dev tools
uv sync --extra wsgi          # For Unix deployments that need uWSGI
```

### Database Management
```bash
uv run flask create-db               # Initialize database tables
uv run flask install                 # Create admin user with secure password
uv run flask reset -e <email/username> -p <password>  # Reset user password
uv run flask add-role -e <email> -r <role>  # Add role to user
```

### Development Server
```bash
uv run flask run                     # Default http://localhost:5000
uv run flask run --port 5001         # Use 5001 locally if 5000 is busy (macOS)
```

### Code Quality
```bash
uv run ruff check .                  # Lint code with ruff
uv run ruff format .                 # Format code with ruff
uv run ruff check --fix .            # Auto-fix linting issues
uv run pre-commit install            # Install pre-commit hooks
```

### Docker Development
```bash
docker compose up --build            # Full stack with Redis, PostgreSQL, Nginx
```

### Internationalization
```bash
uv run flask i18n extract            # Extract translatable strings
uv run flask i18n init <lang>        # Initialize new language
uv run flask i18n update             # Update translations
uv run flask i18n compile            # Compile translations
```

## Flask Architecture Patterns

### Application Factory Pattern

The app is created using the factory pattern in `enferno/app.py`:

```python
from flask import Flask
from enferno.settings import Config
from enferno.extensions import db, cache, mail, session

def create_app(config_object=Config):
    app = Flask(__name__)
    app.config.from_object(config_object)

    register_blueprints(app)
    register_extensions(app)
    register_errorhandlers(app)
    register_commands(app)

    return app
```

### Extension Initialization

Extensions are initialized in `enferno/extensions.py`:

```python
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import DeclarativeBase

class BaseModel(DeclarativeBase):
    pass

db = SQLAlchemy(model_class=BaseModel)
cache = Cache()
mail = Mail()
session = Session()

# Import initialized extensions anywhere:
from enferno.extensions import db, cache, mail
```

### Blueprint Organization

Features are organized into blueprints by functional area:

#### 1. Public Blueprint (`enferno/public/`)
Routes accessible without authentication:

```python
from flask import Blueprint, render_template

public = Blueprint('public', __name__)

@public.route('/')
def index():
    return render_template('public/index.html')
```

#### 2. User Blueprint (`enferno/user/`)
Authentication and user account management:

```python
from flask import Blueprint
from flask_security import auth_required

bp_user = Blueprint('user', __name__)

@bp_user.route('/profile')
@auth_required()
def profile():

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [level09/enferno](https://github.com/level09/enferno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
