---
trigger: always_on
description: Kodosumi is an agentic service framework built on Ray and Litestar.
---

# Kodosumi - LLM Coding Guidelines

## Project Overview
Kodosumi is an agentic service framework built on Ray and Litestar.
- **Framework**: Litestar (ASGI web framework, similar to FastAPI)
- **Frontend**: Jinja2 templates + Beer CSS (Material Design) + Vanilla JavaScript + D3.js
- **Database**: SQLite with SQLAlchemy (admin) and aiosqlite (executions)
- **Deployment**: Ray Serve for distributed execution

## Key Directory Structure
```
kodosumi/
├── service/          # Web service layer
│   ├── app.py       # Main Litestar application factory
│   ├── dashboard.py # Analytics API endpoints
│   └── admin/       # Admin UI (templates, static files)
│       ├── panel.py # Admin routes and views
│       └── templates/
│           ├── _frame.html  # Base template
│           ├── _nav.html    # Navigation sidebar
│           └── dashboard/   # Dashboard templates
├── dtypes.py        # SQLAlchemy models (database schemas)
├── config.py        # Configuration settings
└── spooler.py       # Execution management

data/
├── admin.db         # User database (roles table)
└── execution/       # Execution data
    └── {user_id}/
        └── {exec_id}/
            ├── sqlite3.db  # Execution event log (monitor table)
            ├── in/         # Uploaded files
            └── out/        # Downloaded files
```

## Code Style Guidelines

### Python
- **Async First**: Use async/await for all I/O operations
- **Type Hints**: Required for all function signatures
- **Docstrings**: Required for public functions (Google style)
- **Formatting**: Follow PEP 8
- **Imports**: Group by stdlib, third-party, local

Example:
```python
from typing import Dict, Any, Optional
from pathlib import Path

async def get_execution_data(
    fid: str,
    state: State
) -> Optional[Dict[str, Any]]:
    """Get execution metadata from database.

    Args:
        fid: Execution ID (fid)
        state: Litestar state object

    Returns:
        Metadata dict or None if not found
    """
    ...
```

### Frontend
- **JavaScript**: ES6+ with modules (no build step)
- **CSS**: Beer CSS classes + custom CSS in `<style>` blocks
- **Templates**: Jinja2 with `{% extends "_frame.html" %}`
- **Libraries**: D3.js v7 for visualizations (loaded via CDN)

## Critical Implementation Patterns

### Database Access

#### Execution Databases
```python
# Support both legacy and current naming
db_file = exec_dir / "db.sqlite"
if not db_file.exists():
    db_file = exec_dir / "sqlite3.db"

# Use aiosqlite for execution DBs
async with aiosqlite.connect(db_path) as db:
    db.row_factory = aiosqlite.Row
    async with db.execute(query, params) as cursor:
        rows = await cursor.fetchall()
```

#### Monitor Table Event Types
- **status**: Plain string values ("running", "finished", "error", "awaiting", "starting")
  - ⚠️ NOT JSON - use directly: `message` not `json.loads(message)`
- **meta**: JSON object - requires parsing: `json.loads(message)`
- **error**: Can be plain text or JSON - handle both with try/except
- **inputs**, **result**, **action**, **final**: Usually JSON

Example:
```python
# CORRECT: Status is plain string
status_rows = await query("SELECT message FROM monitor WHERE kind = 'status'")
status = status_rows[0]["message"]  # "finished"

# CORRECT: Meta is JSON
meta_rows = await query("SELECT message FROM monitor WHERE kind = 'meta'")
meta = json.loads(meta_rows[0]["message"])  # {"entry": "agent_name"}

# CORRECT: Error handling for mixed formats
try:
    error_data = json.loads(error_message)
except json.JSONDecodeError:
    error_data = {"error": error_message}
```

### Configuration Access
```python
# CORRECT: Use settings attributes
exec_dir = Path(state["settings"].EXEC_DIR)

# INCORRECT: Don't use non-existent attributes
# data_path = Path(state["settings"].DATA_PATH)  # ❌ Doesn't exist
```

### Litestar Controllers
```python
from litestar import Controller, get
from litestar.datastructures import State

class MyAPI(Controller):
    """API description."""

    tags = ["MyAPI"]  # OpenAPI tags

    @get("/my-endpoint")
    async def my_endpoint(
        self,
        state: State,
        transaction: AsyncSession,  # For admin.db access
        param: Optional[str] = None
    ) -> Dict[str, Any]:
        """Endpoint description."""
        ...
```

Register in `app.py`:
```python
from kodosumi.service.myapi import MyAPI

Router(path="/api/myapi", route_handlers=[MyAPI])
```

### Admin UI Templates
```html
{% extends "_frame.html" %}

{% block styles %}
<style>
/* Component-specific styles */
</style>
{% endblock %}

{% block menu %}
<h3>Page Title</h3>
{% endblock %}

{% block page %}
<div class="active page">
  <!-- Content -->
</div>
{% endblock %}

{% block script %}
<script type="module">
// ES6 JavaScript
</script>
{% endblock %}
```

### Beer CSS Usage
```html
<!-- Grid system -->
<div class="grid">
  <div class="s12 m6 l3">  <!-- 12 cols mobile, 6 tablet, 3 desktop -->
    <article class="border">Content</article>
  </div>
</div>

<!-- Buttons -->
<button class="border">Secondary</button>
<button>Primary</button>
<button class="circle">Round</button>

<!-- Forms -->
<div class="field label suffix border">
  <select>...</select>
  <label>Label Text</label>
  <i>arrow_drop_down</i>
</div>

<!-- Modals -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masumi-network/kodosumi](https://github.com/masumi-network/kodosumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
