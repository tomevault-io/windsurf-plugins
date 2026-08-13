---
trigger: always_on
description: The code and tests MUST NEVER crash, freeze, or consume excessive host resources. All work must implement these safeguards:
---

# Slate - GTK4 Code Editor


## ⚠️ SYSTEM SAFETY MANDATES

The code and tests MUST NEVER crash, freeze, or consume excessive host resources. All work must implement these safeguards:

### Memory Safety
- **ALL tests must have timeouts**: Use `@pytest.mark.timeout(N)` decorator (N ≤ 60s for fast tests, ≤ 120s for slow)
- **Never run full pytest without safeguards** - can freeze system if tests have memory leaks or infinite loops
- **Always use fixtures that cleanup after themselves** - temp directories, orphan processes
- **Test with resource limits**: Use `resource.setrlimit()` to cap memory consumption in test fixtures

### Process Safety
- **Kill orphan processes after each test** - especially GTK apps, subprocess spawns
- **Use fixtures with autouse teardown** to kill leaked processes (e.g., slate, gtk, gdbus)
- **Never leave subprocesses hanging** - always use `finally` or `yield` to terminate
- **E2E/subprocess tests must use `@pytest.mark.slow @pytest.mark.e2e`** - allow exclusion via `-m "not e2e"`

### Test Configuration (Mandatory in pyproject.toml)
```toml
[tool.pytest.ini_options]
timeout = 60
timeout_hard_timeout = 120
```

### Test Safety Rules
1. **Never** run `pytest tests/` without understanding which tests spawn GTK/subprocess
2. **Always** use `--ignore=tests/e2e/` unless you have D-Bus + xvfb + AT-SPI
3. **Safe default**: `pytest tests/core/ tests/services/ -v` (no GTK, no subprocess)
4. **Use markers**: `@pytest.mark.slow`, `@pytest.mark.e2e`, `@pytest.mark.gtk` for filtering

---

## Agent Workflow

### Task Management
- Break complex tasks into smaller steps and track with `todowrite`
- Create todo items with `status: in_progress`, `pending`, or `completed`
- Keep one todo `in_progress` at a time — complete before starting new

### Research & Verification
- Use `websearch` or `codesearch` when uncertain about libraries, APIs, or patterns
- Verify findings by checking existing code in `slate/`, `tests/`, or `pyproject.toml`
- Check system dependencies: `dpkg -l | grep -E "python3-gi|gtk|gtksource"` before assuming packages missing
- Cross-reference with Context7 MCP for framework-specific questions

### Subagent Delegation
- Use `task` tool for parallel work on independent files/components
- Never delegate overlapping file changes to multiple subagents simultaneously
- Before delegating: verify which files the subagent will modify to avoid conflicts
- Combine related file changes in single agent to maintain context

### Quality & Performance
- Run `ruff check` + `ruff format` after every code change
- Run `mypy slate/` before considering work complete
- Test at least one logical path end-to-end (not just unit tests)
- For multi-file changes: verify lint/typecheck pass before moving on

## Commands

```bash
# Development setup
pip install -e ".[dev]"

# Lint & format
ruff check slate/ tests/
ruff format .

# Type check
mypy slate/

# Safe test (core + services only - no GTK/subprocess)
pytest tests/core/ tests/services/ -v

# Full tests (requires xvfb + dbus for E2E)
pytest tests/ -v -m "not e2e"
xvfb-run -a pytest tests/ -v

# Run app
slate                    # open empty editor
slate /path/to/file      # open file on startup
```

## Architecture

```
slate/
├── core/       # Dataclasses, EventBus, plugin API (ZERO GTK imports)
├── services/   # FileService, GitService, ConfigService (lazy GTK imports)
├── ui/         # GTK4 widgets, main window, editor (direct GTK)
└── plugins/    # File explorer, source control (depends only on core)
```

**Layer rules**: `core` → `services` → `ui` → `plugins`. Lower layers never import from higher.

## Critical Anti-Patterns

### Resource Safety (NEVER DO)
- ❌ Never leave orphan processes - always cleanup in fixtures with `finally` or `yield`
- ❌ Never run full pytest without `--ignore=tests/e2e/` unless you have proper display setup
- ❌ Never test GTK without xvfb or display server - will freeze system
- ❌ Never assume tests don't spawn subprocesses - always verify and cleanup

### Code Safety
- ❌ Never `from gi.repository import Gtk` at module level in `core/` or `services/`
- ❌ Never call `gitpython` or `open()` from UI layer — use services
- ❌ Never emit `FileOpenedEvent` directly — only `TabManager` does this
- ❌ Never show windows or mutate tabs in plugin `activate()`
- ❌ Never use GTK signals for cross-component communication — use `EventBus`
- ❌ Never configure `GtkSource.View` directly — use `EditorViewFactory`

### Memory Leak Prevention (T003)
- ❌ **Never set Python instance attributes on GObject wrappers** (`Gtk.*`, `GtkSource.*`, etc.). Setting `self.x = ...` switches PyGObject (≤3.42) into toggle-ref mode, pinning the wrapper + C object alive once the widget is realized → leak. Store state externally (module `WeakKeyDictionary`, `id()`-keyed dict, or locals). Reference fix: `EditorView` in `slate/ui/editor/editor_view.py`.
- ❌ **Never `connect()` a bound method as a signal handler on a signal that outlives the widget** — the closure pins the widget. Connect a closure that derefs `weakref.ref(self)` and no-ops when the view is gone.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [instrumaniak/slate](https://github.com/instrumaniak/slate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
