---
trigger: always_on
description: ReactPy is a Python library for building user interfaces without JavaScript. It creates React-like components that render to web pages using a Python-to-JavaScript bridge.
---

# ReactPy Development Instructions

ReactPy is a Python library for building user interfaces without JavaScript. It creates React-like components that render to web pages using a Python-to-JavaScript bridge.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

**IMPORTANT**: This package uses modern Python tooling with Hatch for all development workflows. Always use Hatch commands for development tasks.

**BUG INVESTIGATION**: When investigating whether a bug was already resolved in a previous version, always prioritize searching through `docs/source/about/changelog.rst` first before using Git history. Only search through Git history when no relevant changelog entries are found.

## Working Effectively

### Bootstrap, Build, and Test the Repository

**Prerequisites:**

-   Install Python 3.9+ from https://www.python.org/downloads/
-   Install Hatch: `pip install hatch`
-   Install Bun JavaScript runtime: `curl -fsSL https://bun.sh/install | bash && source ~/.bashrc`
-   Install Git

**Initial Setup:**

```bash
git clone https://github.com/reactive-python/reactpy.git
cd reactpy
```

**Install Dependencies for Development:**

```bash
# Install core ReactPy dependencies
pip install fastjsonschema requests lxml anyio typing-extensions

# Install ASGI dependencies for server functionality
pip install orjson asgiref asgi-tools servestatic uvicorn fastapi

# Optional: Install additional servers
pip install flask sanic tornado
```

**Build JavaScript Packages:**

-   `hatch run javascript:build` -- takes 15 seconds. NEVER CANCEL. Set timeout to 60+ minutes for safety.
-   This builds three packages: event-to-object, @reactpy/client, and @reactpy/app

**Build Python Package:**

-   `hatch build --clean` -- takes 10 seconds. NEVER CANCEL. Set timeout to 60+ minutes for safety.

**Run Python Tests:**

-   `hatch test --parallel` -- takes 10-30 seconds for basic tests. NEVER CANCEL. Set timeout to 2 minutes for full test suite. **All tests must always pass - failures are never expected or allowed.**
-   `hatch test --parallel --cover` -- run tests with coverage reporting (used in CI)
-   `hatch test --parallel -k test_name` -- run specific tests
-   `hatch test --parallel tests/test_config.py` -- run specific test files

**Run Python Linting and Formatting:**

-   `hatch fmt` -- Run all linters and formatters (~1 second)
-   `hatch fmt --check` -- Check formatting without making changes (~1 second)
-   `hatch fmt --linter` -- Run only linters
-   `hatch fmt --formatter` -- Run only formatters
-   `hatch run python:type_check` -- Run Python type checker (~10 seconds)

**Run JavaScript Tasks:**

-   `hatch run javascript:check` -- Lint and type-check JavaScript (10 seconds). NEVER CANCEL. Set timeout to 30+ minutes.
-   `hatch run javascript:fix` -- Format JavaScript code
-   `hatch run javascript:test` -- Run JavaScript tests

**Interactive Development Shell:**

-   `hatch shell` -- Enter an interactive shell environment with all dependencies installed
-   `hatch shell default` -- Enter the default development environment
-   Use the shell for interactive debugging and development tasks

## Validation

Always manually validate any new code changes through these steps:

**Basic Functionality Test:**

```python
# Add src to path if not installed
import sys, os
sys.path.insert(0, os.path.join("/path/to/reactpy", "src"))

# Test that imports and basic components work
import reactpy
from reactpy import component, html, use_state

@component
def test_component():
    return html.div([
        html.h1("Test"),
        html.p("ReactPy is working")
    ])

# Verify component renders
vdom = test_component()
print(f"Component rendered: {type(vdom)}")
```

**Server Functionality Test:**

```python
# Test ASGI server creation (most common deployment)
from reactpy import component, html
from reactpy.executors.asgi.standalone import ReactPy
import uvicorn

@component
def hello_world():
    return html.div([
        html.h1("Hello, ReactPy!"),
        html.p("Server is working!")
    ])

# Create ASGI app (don't run to avoid hanging)
app = ReactPy(hello_world)
print("✓ ASGI server created successfully")

# To actually run: uvicorn.run(app, host="127.0.0.1", port=8000)
```

**Hooks and State Test:**

```python
from reactpy import component, html, use_state

@component
def counter_component(initial=0):
    count, set_count = use_state(initial)

    return html.div([
        html.h1(f"Count: {count}"),
        html.button({
            "onClick": lambda event: set_count(count + 1)
        }, "Increment")
    ])

# Test component with hooks
counter = counter_component(5)
print(f"✓ Hook-based component: {type(counter)}")
```

**Always run these validation steps before completing work:**

-   `hatch fmt --check` -- Ensure code is properly formatted (never expected to fail)
-   `hatch run python:type_check` -- Ensure no type errors (never expected to fail)
-   `hatch run javascript:check` -- Ensure JavaScript passes linting (never expected to fail)
-   Test basic component creation and rendering as shown above

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactive-python/reactpy](https://github.com/reactive-python/reactpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
