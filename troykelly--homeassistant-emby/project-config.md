---
trigger: always_on
description: A from-scratch Home Assistant integration for Emby Media Server.
---

# Home Assistant Emby Integration

A from-scratch Home Assistant integration for Emby Media Server.

## Project Overview

This is a custom Home Assistant integration that provides media player entities for Emby Media Server clients. It follows Home Assistant 2025 best practices and strict development standards.

## Project Management

### GitHub Project

**ALL work MUST be tracked via GitHub Issues and the Project Board.**

| Item | Value |
|------|-------|
| Project URL | https://github.com/users/troykelly/projects/3 |
| Project Name | Home Assistant Emby Component |
| Repository | troykelly/homeassistant-emby |

### The Iron Law

```
NO CODE CHANGES WITHOUT A LINKED GITHUB ISSUE
```

This is a **VIOLATION**. Every commit, every PR, every change must reference an issue.

**Before writing ANY code:**
1. Check if an issue exists for this work
2. If not, create one
3. Assign yourself and add `status: in-progress`
4. Create branch: `issue-{N}-{description}`
5. Commit with issue reference: `type(scope): message (#N)`
6. Create PR with `Fixes #N` in body

See skill: `ha-emby-github`

### Issue-Driven Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Select Issue ──► ha-emby-issue-selector                     │
│         │                                                       │
│         ├── Bug (unconfirmed) ──► ha-emby-bug-triage            │
│         │                              │                        │
│         │                              ▼                        │
│         │                         Investigate                   │
│         │                              │                        │
│         │                              ▼                        │
│         └── Bug (confirmed) or ──► ha-emby-issue-executor       │
│             Enhancement                │                        │
│                                        ▼                        │
│                                   TDD + PR                      │
│                                        │                        │
│                                        ▼                        │
│                                 Issue auto-closes               │
└─────────────────────────────────────────────────────────────────┘
```

## Mandatory Development Rules

### 1. Test-Driven Development (TDD)

**Every line of code MUST start with a failing test.**

```
RED → GREEN → REFACTOR
```

- Write test first
- Watch it fail (if it passes immediately, your test is wrong)
- Write minimal code to pass
- Refactor while keeping tests green
- No exceptions for "simple" code

See skill: `ha-emby-tdd`

### 2. No `Any` Type

**NEVER use `Any` in type annotations.**

- Use TypedDict for API response structures
- Use dataclasses for internal models
- Use Protocol for interfaces
- Use Generics for containers
- The only exception: `**kwargs: Any` when overriding HA base class methods that require it

See skill: `ha-emby-typing`

### 3. Two Failures = Research

**If code fails twice, STOP and research.**

- Don't guess-and-check
- Read official documentation
- Examine working implementations in HA core
- Understand before attempting again

See skill: `ha-emby-research`

## Project Structure

```
custom_components/embymedia/
├── __init__.py           # Integration setup, async_setup_entry
├── manifest.json         # Integration metadata
├── config_flow.py        # UI configuration flow
├── const.py              # Constants, types, TypedDicts
├── coordinator.py        # DataUpdateCoordinator
├── entity.py             # Base EmbyEntity class
├── api.py                # Emby API client wrapper
├── models.py             # Dataclasses for internal models
│
│   # Entity Platforms
├── media_player.py       # MediaPlayerEntity - playback control
├── remote.py             # RemoteEntity - navigation commands
├── notify.py             # NotifyEntity - on-screen messages
├── button.py             # ButtonEntity - server actions
│
│   # Media Features
├── media_source.py       # MediaSource provider
├── browse.py             # Media browser helpers
├── image.py              # Image proxy for album art
│
│   # Supporting
├── services.py           # Custom services
├── websocket.py          # WebSocket client
├── cache.py              # Response caching
├── exceptions.py         # Custom exceptions
├── diagnostics.py        # Diagnostic download
├── strings.json          # English translations
└── translations/
    └── en.json

tests/
├── conftest.py           # Pytest fixtures
├── test_init.py          # Setup/unload tests
├── test_config_flow.py   # Config flow tests
├── test_coordinator.py   # Coordinator tests
├── test_api.py           # API client tests
├── test_media_player.py  # Media player tests
├── test_remote.py        # Remote entity tests
├── test_notify.py        # Notify entity tests
├── test_button.py        # Button entity tests
└── ...                   # Additional test files
```

## Key Technologies

- **Python 3.12+** - Type hints with modern syntax (`X | None`, not `Optional[X]`)
- **Home Assistant 2025.x** - Latest patterns and APIs
- **pytest + pytest-homeassistant-custom-component** - Testing framework
- **mypy strict** - Type checking
- **aiohttp** - Async HTTP client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [troykelly/homeassistant-emby](https://github.com/troykelly/homeassistant-emby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
