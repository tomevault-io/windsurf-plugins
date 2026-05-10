---
trigger: always_on
description: This document provides comprehensive guidance for Python development in Crescendo Lab, covering architecture patterns, coding style, and best practices for FastAPI applications.
---

# Python Development Guide in CL

This document provides comprehensive guidance for Python development in Crescendo Lab, covering architecture patterns, coding style, and best practices for FastAPI applications.

## Table of Contents
- [Architecture Patterns](#architecture-patterns)
- [Package Structure](#package-structure)
- [Code Organization](#code-organization)
- [Naming Conventions](#naming-conventions)
- [Types and Data Models](#types-and-data-models)
- [Functions and Methods](#functions-and-methods)
- [Error Handling](#error-handling)
- [Async/Await and Concurrency](#asyncawait-and-concurrency)
- [Testing](#testing)
- [Performance Considerations](#performance-considerations)
- [Logging and Observability](#logging-and-observability)
- [Configuration Management](#configuration-management)
- [FastAPI Patterns](#fastapi-patterns)
- [Dependencies and Dependency Injection](#dependencies-and-dependency-injection)
- [Documentation](#documentation)
- [Project Structure](#project-structure)

## Architecture Patterns

### Clean Architecture

Python projects in CL follow a clean architecture pattern with distinct layers:

1. **Entrypoint Layer** (`entrypoint/`)
   - Contains application entry points and configuration
   - HTTP servers, CLI commands, and application bootstrapping
   - Depends on router and domain layers

2. **Domain Layer** (`internal/domain/`)
   - Contains business entities and logic
   - Independent of external frameworks and databases
   - Defines protocols (interfaces) that are implemented by outer layers

3. **Router Layer** (`internal/router/`)
   - HTTP handlers, middleware, and routing logic
   - Connects external HTTP requests to internal domain logic
   - Depends on domain layer

4. **Adapter Layer** (`internal/adapter/`) - *Future implementation*
   - Implements protocols defined in the domain layer
   - Connects the application to external systems (databases, message brokers, etc.)
   - Contains concrete implementations of repositories and services

### Common Patterns

#### Protocol Design
- Define protocols using `typing.Protocol` in the layer that uses them
- Keep protocols small and focused on a single responsibility
- Use dependency injection to provide implementations

```python
from typing import Protocol
from internal.domain.message import Message
from internal.domain.common.error import Error

class MessageRepository(Protocol):
    """Repository for message operations."""
    
    async def save_message(self, message: Message) -> None | Error:
        """Save a message to the repository."""
        ...
    
    async def get_message(self, message_id: str) -> Message | Error:
        """Get a message by ID."""
        ...
```

#### Service Pattern
- Services should be stateless when possible
- Use dependency injection through FastAPI's dependency system
- Services should depend on protocols, not concrete implementations

```python
from typing import Annotated
from fastapi import Depends

class MessageService:
    """Service for message operations."""
    
    def __init__(self, repository: MessageRepository):
        self.repository = repository
    
    async def process_message(self, message: Message) -> None | Error:
        """Process a message."""
        # Business logic here
        return await self.repository.save_message(message)

# Dependency injection
async def get_message_service(
    repository: Annotated[MessageRepository, Depends(get_message_repository)]
) -> MessageService:
    return MessageService(repository)
```

#### Factory Pattern
- Use factories to create complex domain entities or services
- Hide implementation details behind factory protocols

```python
from typing import Protocol
from internal.domain.message import Message, ChannelType

class MessageFactory(Protocol):
    """Factory for creating messages."""
    
    async def create_message(self, channel_type: ChannelType, content: str) -> Message | Error:
        """Create a message for the specified channel type."""
        ...
```

## Package Structure
- Package names should be lowercase with underscores if needed (e.g., `message`, `auto_reply`, `common`)
- Use `__init__.py` files to define package interfaces
- Package name should match the directory name
- Use `internal` directory for code that should not be imported by other projects

## Code Organization

### Package Design
- Keep packages focused on a single responsibility
- Avoid circular dependencies between packages
- Organize code by domain concept, not by technical function

### File Organization
- Keep files to a reasonable size (under 500 lines if possible)
- Group related functionality in the same file
- Place protocols in the same package as the code that uses them
- Use separate files for tests with `test_` prefix

### Imports
- Use absolute imports for internal modules (as per project convention)
- Group imports: standard library, third-party packages, internal packages
- Within each group, imports should be alphabetically sorted
- Use blank lines to separate import groups

```python
import asyncio
from typing import Protocol
from uuid import uuid4

from fastapi import APIRouter, Depends, HTTPException

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chatbotgang/ai-coding-workshop-250712](https://github.com/chatbotgang/ai-coding-workshop-250712) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
