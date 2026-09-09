---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Cursor, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, etc.) when working with code in this repository.

## Rules

Before naming a tool, writing a description, a CHANGELOG entry or a README section, read the rule that covers it - [`rules/README.md`](rules/README.md) indexes them, and each names the test that enforces it. This file covers the rest: architecture, the Tracker API's behaviour, and how to test.

## Project Overview

MCP Yandex Tracker is a Model Context Protocol (MCP) server that provides tools for interacting with Yandex Tracker API. It implements a FastMCP server with protocol-based architecture and optional Redis caching.

## Commands

```bash
task              # Run all checks (format, lint, type checking, tests) - REQUIRED before commits
task format       # Auto-format code
task check        # Run type and format checking
task test         # Run tests
uv sync           # Install dependencies
uv run mcp-tracker # Run the server
```

## Architecture

- **Protocols** (`mcp_tracker/tracker/proto/`): Define API contracts (`QueuesProtocol`, `IssueProtocol`, `GlobalDataProtocol`, `TemplatesProtocol`, `UsersProtocol`, `EntitiesProtocol`, `BoardsProtocol`, `ComponentsProtocol`), each exposed on `AppContext` as `queues` / `issues` / `fields` / `templates` / `users` / `entities` / `boards` / `components`
- **Client** (`mcp_tracker/tracker/custom/client.py`): Implements protocols, handles HTTP requests
- **Caching** (`mcp_tracker/tracker/caching/client.py`): Wraps protocols with Redis caching
- **MCP Server** (`mcp_tracker/mcp/server.py`): Server creation and configuration
- **MCP Tools** (`mcp_tracker/mcp/tools/`): Tool definitions organized by category
  - `_access.py`: Access control helpers (`check_issue_access`, `check_queue_access`)
  - `queue.py` / `queue_write.py`: Queue read-only / write tools
  - `field.py`: Global field and metadata tools (read-only)
  - `template.py`: Issue and comment template tools (read-only)
  - `board.py`: Board and sprint tools (read-only)
  - `component.py` / `component_write.py`: Queue component read-only / write tools (`queue_get_components` itself lives with the queue tools)
  - `issue_read.py` / `issue_write.py`: Issue read-only / write tools
  - `user.py`: User tools (read-only)
  - `__init__.py`: Exports `register_all_tools()` which orchestrates tool registration
  - `*_write.py` modules are only registered when `settings.tracker_read_only=False`
  - project/portfolio/goal modules are only registered when `settings.tracker_entities_enabled=True`
- **Settings** (`mcp_tracker/settings.py`): Pydantic settings from environment variables
- All protocol methods accept optional `auth: YandexAuth | None` parameter for OAuth support.
- All Pydantic models for Yandex Tracker entities inherit from `BaseTrackerEntity`.

### Talking to the Tracker API

- **Reference fields** (`type`, `priority`, `parent`, `sprint`, `followers`, `components`, `project`) use the shared models in `mcp_tracker/tracker/proto/types/inputs.py` (`Issue*Ref`), serialized by `_ref_body()` in the client. How Tracker resolves a bare value is per field, so check before widening a parameter: `type` / `priority` accept an id or a key and resolve a numeric string as an id (verified against the API), `followers` accept a uid or a login the same way, and a 422 from these means the referenced entity does not exist. `components` are the exception - a bare string there is a *name*, which makes a numeric-looking name ambiguous (this is what `components: ["694"]` answered 422 for), hence `IssueComponentRef` requiring exactly one of `id` / `name`. Create and update must accept and send the same value the same way - the API takes a bare key or id on both, so a parameter widened on one has to be widened on the other, or an agent that created an issue with a scalar hits a schema error when it updates the same way.
- **Every request goes through `self._request()`** - or `self._read()`, which is `_request` plus reading the body and is what a method wanting nothing but the body uses. Nothing calls `self._session` directly: the funnel is what builds the auth headers, translates a `TimeoutError` into `TrackerAPITimeout` (`str(TimeoutError())` is the empty string, so an untranslated timeout reaches an agent as a message with nothing in it) and puts every response through `_raise_for_status`, so Tracker's own `errorMessages` / `errors` end up in the raised `TrackerAPIError` instead of a bare "Unprocessable Entity". The funnel exists because these were sixty copies of the same lines and the copies drifted; a method added beside it rather than through it starts that again.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aikts/yandex-tracker-mcp](https://github.com/aikts/yandex-tracker-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
