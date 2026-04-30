---
trigger: always_on
description: An MIT-licensed, open-source AWS emulator built on top of Moto. Runs as a single Docker container on ARM Mac.
---

# Robotocore

An MIT-licensed, open-source AWS emulator built on top of Moto. Runs as a single Docker container on ARM Mac.

## Project Philosophy

- **Free forever**: MIT license, no registration, no telemetry, no paid tiers
- **Drop-in replacement**: Same port (4566), same request routing, same response format as other AWS emulators
- **Built on Moto**: Leverage Moto's ~195 service implementations as the foundation
- **Behavioral fidelity where it matters**: Lambda actually executes, SQS has real visibility timeouts, etc.
- **Single container**: One `docker run` command to get all of AWS locally

## Architecture

```
┌──────────────────────────────────────────────┐
│              Docker Container                │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │     Gateway (port 4566)                │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │  Request Router                  │  │  │
│  │  │  (service detection from headers,│  │  │
│  │  │   URL patterns, query params)    │  │  │
│  │  └──────────┬───────────────────────┘  │  │
│  │             │                          │  │
│  │  ┌──────────▼───────────────────────┐  │  │
│  │  │  Protocol Layer                  │  │  │
│  │  │  (query, json, rest-json,        │  │  │
│  │  │   rest-xml, ec2)                 │  │  │
│  │  └──────────┬───────────────────────┘  │  │
│  │             │                          │  │
│  │  ┌──────────▼───────────────────────┐  │  │
│  │  │  Service Providers               │  │  │
│  │  │  (Moto backends + extensions)    │  │  │
│  │  └──────────┬───────────────────────┘  │  │
│  │             │                          │  │
│  │  ┌──────────▼───────────────────────┐  │  │
│  │  │  In-Memory Stores                │  │  │
│  │  │  (per-account, per-region)       │  │  │
│  │  └─────────────────────────────────-┘  │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

## Project Layout

```
robotocore/
├── CLAUDE.md                  # This file
├── pyproject.toml             # Project config (uv)
├── Dockerfile                 # Single-container build
├── docker-compose.yml         # Dev convenience
├── src/robotocore/
│   ├── __init__.py
│   ├── main.py                # Entrypoint
│   ├── gateway/               # HTTP gateway, request routing
│   │   ├── __init__.py
│   │   ├── app.py             # ASGI/WSGI app
│   │   ├── router.py          # AWS service detection & dispatch
│   │   └── handler_chain.py   # Request/response handler chain
│   ├── protocols/             # AWS protocol parsers/serializers
│   │   ├── __init__.py
│   │   ├── parser.py          # HTTP → Python objects
│   │   └── serializer.py      # Python objects → HTTP response
│   ├── providers/             # Service provider wrappers
│   │   ├── __init__.py
│   │   └── moto_bridge.py     # Bridge to forward requests to moto
│   ├── services/              # Service-specific extensions beyond moto
│   │   └── __init__.py
│   ├── stores/                # In-memory state stores
│   │   └── __init__.py
│   └── utils/                 # Shared utilities
│       └── __init__.py
├── tests/
│   ├── unit/                  # Fast, no-network tests
│   ├── integration/           # Tests against running container
│   └── compatibility/         # Tests that verify AWS parity
├── scripts/
│   ├── dev.py                 # Dev server lifecycle & test runner
│   ├── smoke_test.py          # Cross-service smoke test
│   ├── probe_service.py       # Discover working operations per service
│   └── ...                    # gen_provider, gen_compat_tests, batch_register, etc.
├── vendor/
│   ├── moto/                  # Git submodule: getmoto/moto
│   └── localstack/            # Git submodule (reference implementation)
└── docker/
    └── entrypoint.sh          # Container entrypoint
```

## Development

### Prerequisites

- Python 3.12+
- uv (Python package manager)
- Docker (with ARM/aarch64 support)

### Setup

```bash
git submodule update --init --recursive
uv sync
```

### Running locally (development)

```bash
uv run python -m robotocore.main
```

### Running in Docker

```bash
docker build -t robotocore .
docker run -p 4566:4566 robotocore
```

### Testing

```bash
# Unit tests (parallel, no server needed)
make test                              # or: uv run pytest tests/unit/ -n12

# Compat tests (auto-starts/stops server)
make compat-test                       # or: uv run python scripts/dev.py test-compat

# Compat tests (server already running)
make compat-test-hot

# All tests: unit + compat + integration
make test-all

# Server lifecycle
make start                             # Start dev server in background
make stop                              # Stop dev server
make status                            # Check if server is running
```

## Key Technical Decisions

1. **Gateway on port 4566**: Standard AWS emulator port so existing `aws --endpoint-url` configs work unchanged
2. **Moto as the service layer**: Don't reimplement what Moto already does well. Wrap it, extend it, fix its gaps.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robotocore/robotocore](https://github.com/robotocore/robotocore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
