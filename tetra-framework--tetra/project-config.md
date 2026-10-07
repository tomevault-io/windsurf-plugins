---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tetra is a full-stack component framework for Django using Alpine.js. It combines Python, HTML, CSS, and JavaScript into encapsulated components with server-side state management and inplace DOM updates.

## Common Commands

```bash
make setup        # Install dependencies with uv
make setup-dev    # Install + playwright browsers for E2E tests
make test         # Run pytest
make check        # Run ruff linter
make build-js     # Build JavaScript with esbuild
make build        # Build Python package
make doc          # Build documentation (MkDocs)
make doc-dev      # Serve docs locally
```

## Architecture

### Core Components

- **Component** (`src/tetra/components/base.py`): Base class combining Python logic, HTML template, CSS, and JavaScript
- **BasicComponent**: Simplified component without form handling
- **FormComponent**: Extends Component with Django form capabilities
- **ModelFormComponent**: Django ModelForm integration (needs form_class + object)
- **ReactiveComponent** (`src/tetra/components/reactive.py`): WebSocket-based reactive updates (requires Django Channels)
- **ReactiveModel** (`src/tetra/models.py`): Abstract model that broadcasts changes to components via WebSocket

### State Management

- **Resumable server state**: Encrypted state preserved between method calls
- **Public shared state**: Exposed to frontend via Alpine.js data bindings using `@public` decorator
- **Protocol**: Custom `tetra-1.0` protocol using HTTP POST for method calls and WebSockets for reactive updates

### Component Libraries & File Structure

Every component belongs to a **Library**. Libraries are discovered automatically from `<app>.components.<library_name>`.

#### Inline components (simple, small)
```
myapp/
  components/
    default.py          ← Put multiple component classes directly here
    widgets.py          ← Another library named "widgets"
```

#### Directory-style components (with separate HTML/JS/CSS files)
```
myapp/
  components/
    default/
      __init__.py       ← Can contain inline component classes
      my_calendar/
        __init__.py     ← Component class (ONE per directory)
        my_calendar.html  ← External template
        my_calendar.js    ← External JavaScript
        my_calendar.css   ← External CSS
```

**Rule:** Directory-style component modules may contain only ONE component class. Inline library modules can contain multiple.

### Django Integration

1. Add `'tetra'` to `INSTALLED_APPS`
2. Add `'tetra.middleware.TetraMiddleware'` to `MIDDLEWARE`
3. Include Tetra URLs: `path("", include("tetra.urls"))`
4. Use `{% load tetra %}` template tag
5. Render with `{% ComponentClassName / %}` for default library or `{% library.ComponentClassName / %}`

### JavaScript/Build Pipeline

- JavaScript source in `src/tetra/js/`
- Built assets in `src/tetra/static/tetra/js/`
- Uses esbuild for bundling
- Alpine.js with Morph plugin for inplace updates

### Testing

- pytest with pytest-django
- E2E tests use pytest-playwright
- Test apps in `tests/apps/`

---

## Creating Components

### Minimal Component

```python
from tetra import Component, public

class Counter(Component):
    count = public(0)

    @public
    def increment(self):
        self.count += 1

    template = """
    <div>
        <span x-text="count"></span>
        <button @click="increment()">+1</button>
    </div>
    """
```

### Full-Featured Component

```python
from sourcetypes import django_html, javascript, css
from tetra import Component, public

class TodoItem(Component):
    # Public attributes — available in both Python and Alpine.js
    title = public("")
    done = public(False)

    # Private attributes — server-only, saved with state
    todo = None

    def load(self, todo, *args, **kwargs):
        """Called on init AND after state resume."""
        self.todo = todo
        self.title = todo.title
        self.done = todo.done

    @public.watch("title", "done").debounce(200)
    def save(self, value, old_value, attr):
        """Auto-called when watched public attrs change."""
        self.todo.title = self.title
        self.todo.done = self.done
        self.todo.save()

    @public(update=False)
    def delete_item(self):
        """update=False skips re-render."""
        self.todo.delete()
        self.client._removeComponent()

    template: django_html = """
    <div>
        <input type="checkbox" x-model="done">
        <input type="text" x-model="title">
    </div>
    """

    script: javascript = """
    export default {
        init() { ... }
    }
    """

    style: css = """
    .todo-strike { text-decoration: line-through; }
    """
```

---

## Key Concepts

### `public()` — Public Attributes

```python
name = public("default")       # Available in Python AND Alpine.js
count = public(0)
items = public([])
```

Values must be JSON-serializable (extended: `datetime`, `date`, `time`, `set`, `Decimal`).

### `@public` — Public Methods

```python
@public                              # Re-renders component after call (default)
def my_method(self): ...

@public(update=False)                # No re-render

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tetra-framework/tetra](https://github.com/tetra-framework/tetra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
