---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Allora Network Python SDK is a comprehensive async client library for interacting with the Allora blockchain network. It provides both HTTP API access and direct blockchain interaction capabilities including transaction submission and real-time event subscriptions.

### Core Architecture

- **Protobuf Client**: `src/allora_sdk/protobuf_client/client.py` contains the main `ProtobufClient` class for blockchain interaction
- **HTTP API Client**: `src/allora_sdk/api_client_v2.py` provides REST API access for queries and data retrieval  
- **Event Subscriptions**: WebSocket-based real-time event streaming with typed protobuf message support
- **Transaction Support**: Full transaction building and submission for worker payload inference submission
- **Chain Support**: Supports testnet, mainnet, and local development chains

### Key Components

#### Blockchain Interaction (`protobuf_client`)
- `ProtobufClient`: Main blockchain client with transaction, query, and event capabilities
- `AlloraTransactions`: Transaction builder supporting worker payload submissions and standard Cosmos operations
- `AlloraWebsocketSubscriber`: Real-time event subscription with both generic and typed protobuf callbacks
- `EventRegistry`: Auto-discovers protobuf Event classes for type-safe event handling
- `AlloraQueries`: Blockchain query interface (queries, balances, account info, etc.)

#### HTTP API (`api_client_v2`)
- `AlloraAPIClient`: HTTP client for topics, inferences, and price predictions
- `AlloraTopic`: Model for network topics with metadata
- `AlloraInference`: Model for inference data with confidence intervals

#### Protobuf Integration
- Full Allora protobuf message support (emissions v1-v9)
- betterproto-based message serialization with cosmpy integration
- Type-safe event marshaling from JSON to protobuf instances

## Common Development Commands

### Testing
```bash
# Run all tests across Python versions
tox

# Run tests for specific Python version
tox -e 3.12

# Run tests directly with pytest
pytest tests/

# Run specific test file
pytest tests/test_api_client_unit.py

# Run integration tests (requires API access)
pytest tests/test_api_client_integration.py
```

### Linting and Type Checking
```bash
# Run linting (black formatter)
tox -e lint

# Run type checking
tox -e type

# Format code manually
black .

# Type check manually
mypy src tests
```

### Building and Installation
```bash
# Install in development mode
pip install -e .

# Install with dev dependencies
pip install -e .[dev]

# Build wheel
python -m build
```

## Testing Strategy

The project uses a dual testing approach:

1. **Unit Tests** (`test_api_client_unit.py`): Mock-based tests using custom `StarletteMockFetcher` that simulates API responses
2. **Integration Tests** (`test_api_client_integration.py`): Real API tests against testnet (requires network access)

The mock testing framework in `tests/mock_data.py` provides a `MockServer` class that can simulate API responses and pagination scenarios.


## Coding Style Guidelines

### Structure and Control Flow
- **Minimize indentation**: Prefer early returns over nested blocks. Avoid nested try/catch and nested if statements.
- **Guard clauses**: Use guard clauses at the beginning of functions to handle edge cases and exit early.
- **Single responsibility**: Each function should have one clear purpose.

```python
# Preferred - early returns, minimal nesting
def process_events(events: List[Dict[str, Any]]) -> List[ProcessedEvent]:
    if not events:
        return []
    
    filtered = [ e for e in events if e.get("type") == "target_type" ]
    if not filtered:
        return []
    
    return [ ProcessedEvent.from_dict(e) for e in filtered ]

# Avoid - nested blocks
def process_events_bad(events: List[Dict[str, Any]]) -> List[ProcessedEvent]:
    if events:
        filtered = []
        for e in events:
            if e.get("type") == "target_type":
                filtered.append(e)
        if filtered:
            results = []
            for e in filtered:
                results.append(ProcessedEvent.from_dict(e))
            return results
    return []
```

### Comprehensions and Data Processing
- **Use comprehensions liberally**: Even multiple comprehensions in succession for complex transformations.
- **Chain comprehensions**: It's acceptable to do multiple passes for clarity.

```python
# Preferred - clear, functional style, using same list variable name
events = websocket_data.get("events", [])
events = [ e for e in events if e.get("type") == target_type ]
events = [ marshal_event(e) for e in events ]
events = [ e for e in events if e is not None ]
```

### Type Safety and Documentation
- **Strong typing always**: Use type hints on all functions, variables, and class attributes.
- **Prefer structured types**: Use dataclasses, Pydantic models, or NamedTuple over dictionaries.
- **Convert external data**: Transform incoming dictionaries from external packages into typed objects.
- **Use enums**: For constants and fixed sets of values.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allora-network/allora-sdk-py](https://github.com/allora-network/allora-sdk-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
