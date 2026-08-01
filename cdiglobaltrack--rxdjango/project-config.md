---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RxDjango is a real-time integration layer between Django and React that provides seamless bidirectional communication via WebSockets. It sits on top of Django Channels and Django REST Framework, automatically synchronizing backend state with React frontend using a nested serializer-based architecture.

**Key architectural principles:**
- Serializers define both REST API structure AND real-time state
- Instances are flattened and cached (MongoDB), then rebuilt on client
- Django signals trigger automatic cache updates and WebSocket broadcasts
- TypeScript interfaces are auto-generated from Django serializers
- Each `ContextChannel` creates a stateful WebSocket connection for a specific data context

## Development Commands

### Python/Django Backend

Build and install the package:
```bash
pip install -e .                      # Install in development mode (auto-compiles C extensions)
# Or build for distribution:
python -m build                       # Creates wheel and sdist in dist/
```

Generate frontend TypeScript files:
```bash
python manage.py makefrontend          # Generate all TS interfaces and channels
python manage.py makefrontend --dry-run   # Preview changes without writing
python manage.py makefrontend --force     # Force rebuild all files
python manage.py runserver --makefrontend # Auto-regenerate on changes during dev
```

Other commands:
```bash
python manage.py broadcast_system_message  # Broadcast to all connected clients
```

### React/TypeScript Frontend

Located in `rxdjango-react/`:
```bash
cd rxdjango-react
npm install              # or yarn install
npm run build           # Build TypeScript to dist/
npm run dev             # Watch mode for development
npm test                # Run Jest tests
npm run test:watch      # Watch mode for tests
npm run test:debug      # Debug tests with Node inspector
```

### Testing

**Frontend tests** (Jest):
```bash
cd rxdjango-react
yarn test --ci       # Run all tests (CI mode, no watch)
npm test             # Interactive/watch mode
```

**Frontend lint** (ESLint + TypeScript):
```bash
cd rxdjango-react
npm run lint
```

**Backend unit tests** (no external services needed):
```bash
pytest rxdjango/tests/
```

**Backend integration tests** (requires Redis on port 6379 and MongoDB on port 27017):
```bash
cd test_project
python manage.py test react_test.tests
python manage.py test react_test.tests.test_write_operations  # Write operation tests
```

**Backend lint** (flake8):
```bash
flake8 --ignore=E501,W504 rxdjango
```

### Building Distribution

```bash
python -m build  # Creates both wheel (.whl) and source distribution (.tar.gz) in dist/
```

Note: Requires the `build` package: `pip install build`

## Architecture Overview

### Core Components

**ContextChannel** (`rxdjango/channels.py`)
- The main API surface. Subclass this to create a real-time channel
- Each ContextChannel has a `Meta.state` that points to a ModelSerializer
- Metaclass `ContextChannelMeta` introspects the serializer and builds:
  - `StateModel`: Tree representing nested serializer structure
  - `WebsocketRouter`: Routes updates to connected clients
  - `SignalHandler`: Connects Django model signals to cache/broadcast system

**StateModel** (`rxdjango/state_model.py`)
- Recursively analyzes nested serializers to build a dependency tree
- Tracks relationships (ForeignKey, ManyToMany, OneToOne, custom properties)
- Each layer knows its `instance_type` (serializer path), `anchor_key` (query path), and children
- Provides `serialize_instance()` and `serialize_state()` for flattening instances
- Automatically exports TypeScript interfaces via `export_interface()`

**SignalHandler** (`rxdjango/signal_handler.py`)
- Monkey-patches `Model.save_base()` to attach `RxMeta` for tracking parent changes
- Connects to Django's `pre_save`, `post_save`, `pre_delete`, `post_delete` signals
- On model changes: serializes instance → writes to Mongo → broadcasts to WebSocket clients
- Handles parent relationship changes (e.g., moving an object to a different parent context)
- Uses `transaction.on_commit()` to ensure atomic updates

**StateConsumer** (`rxdjango/consumers.py`)
- AsyncWebsocketConsumer that handles WebSocket lifecycle
- Authenticates via `rest_framework.authtoken.models.Token`
- Loads initial state via `StateLoader`, then subscribes to real-time updates
- Supports `@action` decorated methods (RPC from frontend to backend)
- Supports `@consumer` decorated methods (subscribe to Django Channels groups)

**Caching System**
- **MongoDB** (`rxdjango/mongo.py`): Persistent cache of flattened instances with optimistic locking
- **Redis** (`rxdjango/redis.py`): Coordinates cooldowns and transient state
- Cache is cleared on `python manage.py migrate` via `post_migrate` signal

### TypeScript Generation

**Interface Export** (`rxdjango/ts/interfaces.py`)
- Introspects serializer fields and generates TypeScript interfaces
- Handles nested serializers, lists, optional fields, and type mappings
- Uses decorators like `@extend_ts()` to add custom TS properties

**Channel Export** (`rxdjango/ts/channels.py`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CDIGlobalTrack/rxdjango](https://github.com/CDIGlobalTrack/rxdjango) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
