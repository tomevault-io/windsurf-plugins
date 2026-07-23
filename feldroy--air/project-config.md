---
trigger: always_on
description: Air is a Python web framework built on FastAPI/Starlette. HTML-first, zero config. This file tells you everything you need to build a complete Air app from scratch.
---

# Building Apps with Air

Air is a Python web framework built on FastAPI/Starlette. HTML-first, zero config. This file tells you everything you need to build a complete Air app from scratch.

## Working from HTML Mockups

If the user doesn't have mockups yet, use the `/frontend-design` skill to create static HTML files in `mockups/`. Then follow this workflow starting at step 2.

When the user provides static HTML files as mockups, follow this workflow:

1. **Archive the originals.** If the HTML files aren't already in `mockups/`, move them there so the raw design is preserved as a reference. Delete any copies left behind (e.g. at the project root). Create the directory if it doesn't exist.

2. **Copy to templates/.** Copy each mockup from `mockups/` into `templates/`. This is where Air's Jinja renderer looks for templates. The originals stay in `mockups/` as a reference. Create the directory if it doesn't exist.

3. **Template them.** Convert the static HTML into Jinja templates:
   - Extract repeated structure (nav, footer, head) into a `base.html` with `{% block %}` tags
   - Replace hardcoded text with `{{ variables }}` where the content should come from Python
   - Replace hardcoded URLs with route references
   - Keep `href="/static/..."` paths as-is (Air serves static files at `/static/` automatically)
   - Add `{% extends "base.html" %}` and `{% block content %}` to page templates

4. **Wire up routes.** In `main.py`, use `app.jinja` (auto-created, points at `templates/`) to render:

```python
import air

app = air.Air()

@app.page
def index(request: air.Request):
    return app.jinja(request, "index.html", title="Home")

@app.page
def about(request: air.Request):
    return app.jinja(request, "about.html", title="About")
```

5. **Move static assets.** If the mockups reference CSS, JS, or images, move those into `static/` and update the paths in templates to use `/static/...`.

### Example: mockup to working app

Given a file `landing.html`:

```
mockups/
  landing.html         <- original, untouched
templates/
  base.html            <- extracted layout
  landing.html         <- templated version
static/
  css/style.css        <- extracted from mockup
main.py                <- routes using app.jinja
pyproject.toml
```

## Minimal App

```python
import air

app = air.Air()

@app.page
def index():
    return air.H1("Hello, World!")
```

Run: add `[tool.fastapi]` to `pyproject.toml`, then `air run` or `uv run air run`.

```toml
[tool.fastapi]
app = "main:app"
```

That's it. No settings file, no config module, no URL dispatcher files. Air reads everything from decorators and directory conventions (`static/`, `templates/`), so there's nothing to configure in a separate file. Customization happens through composition: add routers with `app.include_router()`, add middleware, pass parameters to `air.Air()`. Don't subclass `Air` or `AirRouter`.

## Routing

### @app.page: function name becomes the URL

```python
@app.page
def index():          # route: /
    return air.H1("Home")

@app.page
def about_us():       # route: /about-us (underscores become dashes)
    return air.H1("About")

@app.page
def contact():        # route: /contact
    return air.H1("Contact")
```

### Explicit paths with HTTP method decorators

```python
@app.get("/users/{user_id}")
def get_user(user_id: int) -> air.Div:
    return air.Div(air.H1(f"User {user_id}"))

@app.post("/submit")
async def submit(request: air.Request) -> air.Div:
    form_data = await request.form()
    return air.Div(air.P("Received"))
```

All standard HTTP methods: `app.get()`, `app.post()`, `app.put()`, `app.patch()`, `app.delete()`.

### async def vs def

Use `async def` when the handler calls `await` (e.g. `await request.form()`, `await request.json()`). Use plain `def` for everything else. Both work in all cases, but mixing `await` into a `def` route is a syntax error, and using `async def` without `await` wastes no resources but is unnecessary.

### Reverse URL resolution

Every decorated route gets a `.url()` method automatically:

```python
get_user.url(user_id=42)                           # "/users/42"
get_user.url(user_id=42, query_params={"tab": 1})  # "/users/42?tab=1"
index.url()                                         # "/"
```

Use in templates and tags: `air.A("Home", href=index.url())`.

### Routers for multi-file apps

```python
# views.py
import air

router = air.AirRouter()

@router.page
def dashboard():
    return air.H1("Dashboard")

# main.py
import air
from views import router

app = air.Air()
app.include_router(router)
```

## HTML Tags (Air Tags)

Air represents HTML as Python objects. Every standard HTML element is available as `air.TagName`.

```python
air.Div(
    air.H1("Welcome"),
    air.P("Hello ", air.Strong("world")),
    air.A("Click here", href="/about"),
    class_="container",
    id_="main",
)
```

Renders to: `<div class="container" id="main"><h1>Welcome</h1><p>Hello <strong>world</strong></p><a href="/about">Click here</a></div>`

### Air Tag Attribute rules

- Python reserved words use trailing underscore: `class_`, `for_`, `type_`, `id_`, `async_`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feldroy/air](https://github.com/feldroy/air) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
