---
trigger: always_on
description: Enforce best practices for FastAPI MVC architecture with TailwindCSS, SQLite ORM, Fly.io deployment, and GitHub version control.
---


# FastAPI MVC Development Guide

## 🧱 Architecture & Project Structure

**Description**: Standard MVC organization for FastAPI projects with clear separation of concerns.

### Project Structure
```
project/
├── app/
│   ├── __init__.py
│   ├── models/          # Database models (SQLAlchemy/SQLModel)
│   ├── controllers/     # Business logic layer
│   ├── routes/          # API route definitions
│   ├── templates/       # Jinja2 templates with TailwindCSS
│   ├── utils/           # Utility functions
│   └── database.py      # Database configuration
├── tests/               # Test files
├── migrations/          # Alembic migrations
├── main.py             # FastAPI application entry point
├── requirements.txt    # Dependencies
├── fly.toml            # Fly.io deployment config
└── README.md           # Project documentation
```

### File Responsibilities
- **Models** (`app/models/`): SQLAlchemy/SQLModel database models
- **Controllers** (`app/controllers/`): Business logic and data access operations
- **Routes** (`app/routes/`): HTTP request/response handling and route definitions
- **Templates** (`app/templates/`): Jinja2 templates with TailwindCSS styling
- **Utils** (`app/utils/`): Shared utility functions and helpers

---

## ⚙️ FastAPI Best Practices

**Description**: Core FastAPI patterns for application setup, routing, and dependency injection.

### Application Setup
Use lifespan context managers for startup/shutdown logic.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Manage application lifespan."""
    await init_db()  # Startup
    yield
    # Shutdown

app = FastAPI(
    title="API Title",
    description="API Description", 
    version="1.0.0",
    lifespan=lifespan
)

app.mount("/static", StaticFiles(directory="app/static"), name="static")
templates = Jinja2Templates(directory="app/templates")
```

### Route Organization
Organize routes by feature using `APIRouter` with OpenAPI tags.

```python
from fastapi import APIRouter

router = APIRouter(prefix="/api/v1", tags=["todos"])

@router.post("/todos", response_model=TodoResponse, tags=["todos"])
async def create_todo_endpoint(
    todo: TodoCreate,
    db: AsyncSession = Depends(get_db)
) -> TodoResponse:
    """Create a new todo."""
    return await create_todo(db, todo)
```

### Dependency Injection
- Use `Depends()` for database connections, authentication, and services
- Create reusable dependencies for common operations
- Use async dependencies for I/O operations

---

## 🎨 Frontend Best Practices

**Description**: TailwindCSS + Vanilla JavaScript frontend patterns with mobile-first responsive design.

### Template Structure
Use Jinja2 templates with FastAPI for server-side rendering.

```python
from fastapi import Request
from fastapi.templating import Jinja2Templates
from fastapi.responses import HTMLResponse

templates = Jinja2Templates(directory="app/templates")

@app.get("/", response_class=HTMLResponse)
async def read_root(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})
```

### Base Template
Standard base template with TailwindCSS CDN.

```html
<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}{{ title or "FastAPI App" }}{% endblock %}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    {% block head %}{% endblock %}
</head>
<body class="h-full bg-gray-50">
    <div class="min-h-full">
        {% block content %}{% endblock %}
    </div>
    {% block scripts %}{% endblock %}
</body>
</html>
```

### HTML Best Practices
- Use semantic HTML5 elements (`<header>`, `<main>`, `<section>`, etc.)
- Include proper meta tags for SEO and responsive design
- Use `lang` attribute on `<html>` element
- Implement proper heading hierarchy (h1 → h2 → h3)
- Use descriptive alt text for images
- Include ARIA labels for accessibility

### TailwindCSS Guidelines
- Use utility-first approach with consistent spacing scale
- Leverage responsive prefixes (`sm:`, `md:`, `lg:`, `xl:`)
- Use CSS Grid and Flexbox utilities for layouts
- Use state variants (`hover:`, `focus:`, `active:`, `disabled:`)
- Optimize for mobile-first responsive design

### Mobile-First Design
**Core Principles**: Design for mobile first, then enhance for larger screens.

**Responsive Breakpoints**:
- Base: 320px+ (mobile)
- `sm:`: 640px+ (tablet)
- `md:`: 768px+ (small desktop)
- `lg:`: 1024px+ (desktop)
- `xl:`: 1280px+ (large desktop)

**Mobile-First Patterns**:
```html
<!-- Responsive grid -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
  <!-- Content adapts from 1 column on mobile to 4 on xl screens -->
</div>

<!-- Responsive typography -->
<h1 class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">
  Responsive Heading
</h1>

<!-- Responsive spacing -->
<div class="p-4 sm:p-6 md:p-8 lg:p-12">
  <!-- Padding increases with screen size -->
</div>
```

**Mobile Considerations**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/winnwy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
