---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Browser-Use is an async python >= 3.11 library that implements AI browser driver abilities using LLMs + CDP (Chrome DevTools Protocol). The core architecture enables AI agents to autonomously navigate web pages, interact with elements, and complete complex tasks by processing HTML and making LLM-driven decisions.

## High-Level Architecture

The library follows an event-driven architecture with several key components:

### Core Components

- **Agent (`browser_use/agent/service.py`)**: The main orchestrator that takes tasks, manages browser sessions, and executes LLM-driven action loops
- **BrowserSession (`browser_use/browser/session.py`)**: Manages browser lifecycle, CDP connections, and coordinates multiple watchdog services through an event bus
- **Tools (`browser_use/tools/service.py`)**: Action registry that maps LLM decisions to browser operations (click, type, scroll, etc.)
- **DomService (`browser_use/dom/service.py`)**: Extracts and processes DOM content, handles element highlighting and accessibility tree generation
- **LLM Integration (`browser_use/llm/`)**: Abstraction layer supporting OpenAI, Anthropic, Google, Groq, and other providers

### Event-Driven Browser Management

BrowserSession uses a `bubus` event bus to coordinate watchdog services:
- **DownloadsWatchdog**: Handles PDF auto-download and file management
- **PopupsWatchdog**: Manages JavaScript dialogs and popups
- **SecurityWatchdog**: Enforces domain restrictions and security policies
- **DOMWatchdog**: Processes DOM snapshots, screenshots, and element highlighting
- **AboutBlankWatchdog**: Handles empty page redirects

### CDP Integration

Uses `cdp-use` (https://github.com/browser-use/cdp-use) for typed CDP protocol access. All CDP client management lives in `browser_use/browser/session.py`.

We want our library APIs to be ergonomic, intuitive, and hard to get wrong.

## Development Commands

**Setup:**
```bash
uv venv --python 3.11
source .venv/bin/activate
uv sync
```

**Testing:**
- Run CI tests: `uv run pytest -vxs tests/ci`
- Run all tests: `uv run pytest -vxs tests/`
- Run single test: `uv run pytest -vxs tests/ci/test_specific_test.py`

**Quality Checks:**
- Type checking: `uv run pyright`
- Linting/formatting: `uv run ruff check --fix` and `uv run ruff format`
- Pre-commit hooks: `uv run pre-commit run --all-files`

**MCP Server Mode:**
The library can run as an MCP server for integration with Claude Desktop:
```bash
uvx browser-use[cli] --mcp
```

## Code Style

- Use async python
- Use tabs for indentation in all python code, not spaces
- Use the modern python >3.12 typing style, e.g. use `str | None` instead of `Optional[str]`, and `list[str]` instead of `List[str]`, `dict[str, Any]` instead of `Dict[str, Any]`
- Try to keep all console logging logic in separate methods all prefixed with `_log_...`, e.g. `def _log_pretty_path(path: Path) -> str` so as not to clutter up the main logic.
- Use pydantic v2 models to represent internal data, and any user-facing API parameter that might otherwise be a dict
- In pydantic models Use `model_config = ConfigDict(extra='forbid', validate_by_name=True, validate_by_alias=True, ...)` etc. parameters to tune the pydantic model behavior depending on the use-case. Use `Annotated[..., AfterValidator(...)]` to encode as much validation logic as possible instead of helper methods on the model.
- We keep the main code for each sub-component in a `service.py` file usually, and we keep most pydantic models in `views.py` files unless they are long enough deserve their own file
- Use runtime assertions at the start and end of functions to enforce constraints and assumptions
- Prefer `from uuid_extensions import uuid7str` +  `id: str = Field(default_factory=uuid7str)` for all new id fields
- Run tests using `uv run pytest -vxs tests/ci`
- Run the type checker using `uv run pyright`

## CDP-Use

We use a thin wrapper around CDP called cdp-use: https://github.com/browser-use/cdp-use. cdp-use only provides shallow typed interfaces for the websocket calls, all CDP client and session management + other CDP helpers still live in browser_use/browser/session.py.

- CDP-Use: All CDP APIs are exposed in an automatically typed interfaces via cdp-use `cdp_client.send.DomainHere.methodNameHere(params=...)` like so:
  - `cdp_client.send.DOMSnapshot.enable(session_id=session_id)`
  - `cdp_client.send.Target.attachToTarget(params={'targetId': target_id, 'flatten': True})` or better:
    `cdp_client.send.Target.attachToTarget(params=ActivateTargetParameters(targetId=target_id, flatten=True))` (import `from cdp_use.cdp.target import ActivateTargetParameters`)
  - `cdp_client.register.Browser.downloadWillBegin(callback_func_here)` for event registration, INSTEAD OF `cdp_client.on(...)` which does not exist!

## Keep Examples & Tests Up-To-Date

- Make sure to read relevant examples in the `examples/` directory for context and keep them up-to-date when making changes.
- Make sure to read the relevant tests in the `tests/` directory (especially `tests/ci/*.py`) and keep them up-to-date as well. 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [browser-use/browser-use](https://github.com/browser-use/browser-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
