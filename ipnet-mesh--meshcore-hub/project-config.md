---
trigger: always_on
description: This document provides context and guidelines for AI coding assistants working on the MeshCore Hub project.
---

# AGENTS.md - AI Coding Assistant Guidelines

This document provides context and guidelines for AI coding assistants working on the MeshCore Hub project.

## Agent Rules

### Critical Rules (MUST follow)

* **Always use parenthesized exception tuples** — `except (ValueError, TypeError):` not `except ValueError, TypeError:`. The comma form is Python 2 syntax and will fail at import time in Python 3. This is the most common error that passes visual review but breaks the application.
* You MUST use Python (version in `.python-version` file)
* You MUST activate a Python virtual environment in the `.venv` directory or create one if it does not exist:
  - `ls ./.venv` to check if it exists
  - `python -m venv .venv` to create it
* You MUST always activate the virtual environment before running any commands
  - `source .venv/bin/activate`
* You MUST install all project dependencies using `pip install -e ".[dev]"` command`
* You MUST install `pre-commit` for quality checks
* **Never `git push` without explicit confirmation** — staging and committing after discrete changes is fine, but pushing to remote requires the user to explicitly request it
* You MUST keep project documentation in sync with behavior/config/schema changes made in code (at minimum update relevant sections in `README.md`, `SCHEMAS.md`, `docs/upgrading.md`, `docs/letsmesh.md` when applicable)
* Before commiting:
  - Run **targeted tests** for the components you changed, not the full suite:
    - `pytest tests/test_web/` for web-only changes (templates, static JS, web routes)
    - `pytest tests/test_api/` for API changes
    - `pytest tests/test_collector/` for collector changes
    - `pytest tests/test_common/` for common models/schemas/config changes
    - Only run the full `pytest` if changes span multiple components
  - Run `pre-commit run --all-files` to perform all quality checks

## Project Overview

MeshCore Hub is a Python 3.14+ monorepo for managing and orchestrating MeshCore mesh networks. Data ingestion is done via [meshcore-packet-capture](https://github.com/agessaman/meshcore-packet-capture), which captures MeshCore mesh traffic and publishes events to MQTT. MeshCore Hub then collects, stores, and presents this data. It consists of four main components:

- **meshcore_collector**: Collects MeshCore events from MQTT and stores them in a database
- **meshcore_api**: REST API for querying data
- **meshcore_web**: Web dashboard for visualizing network status
- **meshcore_common**: Shared utilities, models, and configurations

## Key Documentation

- [SCHEMAS.md](SCHEMAS.md) - MeshCore event JSON schemas and database mappings
- [docs/upgrading.md](docs/upgrading.md) - Upgrade guide for breaking changes
- [docs/letsmesh.md](docs/letsmesh.md) - LetsMesh packet decoding details
- [docs/hosting/nginx-proxy-manager.md](docs/hosting/nginx-proxy-manager.md) - Nginx Proxy Manager admin setup
- [docs/seeding.md](docs/seeding.md) - Seed data format and import guide
- [docs/i18n.md](docs/i18n.md) - Translation reference guide

## Technology Stack

| Category | Technology |
|----------|------------|
| Language | Python 3.14+ |
| Package Management | pip with pyproject.toml |
| CLI Framework | Click |
| Configuration | Pydantic Settings |
| Database ORM | SQLAlchemy 2.0 (async) |
| Migrations | Alembic |
| REST API | FastAPI |
| MQTT Client | paho-mqtt |
| MQTT Broker | [meshcore-mqtt-broker](https://github.com/michaelhart/meshcore-mqtt-broker) (WebSocket + JWT auth) |
| Templates | Jinja2 (server), lit-html (SPA) |
| Frontend | ES Modules SPA with client-side routing |
| CSS Framework | Tailwind CSS v4 (CLI) + DaisyUI v5 |
| Frontend Build | Node.js 22 LTS (`npm run build`) |
| Testing | pytest, pytest-asyncio |
| Formatting | black |
| Linting | flake8 |
| Type Checking | mypy |

## Code Style Guidelines

### General

- Follow PEP 8 style guidelines
- Use `black` for code formatting (line length 88)
- Use type hints for all function signatures
- Write docstrings for public modules, classes, and functions
- Keep functions focused and under 50 lines where possible

### Imports

```python
# Standard library
import os
from datetime import datetime
from typing import Optional

# Third-party
from fastapi import FastAPI, Depends
from pydantic import BaseModel
from sqlalchemy import select

# Local
from meshcore_hub.common.config import Settings
from meshcore_hub.common.models import Node
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Modules | snake_case | `node_tags.py` |
| Classes | PascalCase | `NodeTagCreate` |
| Functions | snake_case | `get_node_by_key()` |
| Constants | UPPER_SNAKE_CASE | `DEFAULT_MQTT_PORT` |
| Variables | snake_case | `public_key` |
| Type Variables | PascalCase | `T`, `NodeT` |

### Pydantic Models

```python
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional

class NodeRead(BaseModel):
    """Schema for reading node data from API."""

    id: str
    public_key: str = Field(..., min_length=64, max_length=64)
    name: Optional[str] = None
    adv_type: Optional[str] = None
    last_seen: Optional[datetime] = None

    model_config = {"from_attributes": True}
```

### SQLAlchemy Models

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ipnet-mesh/meshcore-hub](https://github.com/ipnet-mesh/meshcore-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
