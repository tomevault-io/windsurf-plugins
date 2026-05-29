---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Waku is a modular, type-safe Python framework (3.11+) inspired by NestJS, built on dishka IoC container. It provides:
- Modular architecture with dependency injection
- CQRS implementation (commands, queries, events) with message bus pattern
- Event sourcing with projections, snapshots, and upcasting
- Extension system for lifecycle hooks and custom behaviors
- Framework-agnostic design (works with FastAPI, Litestar, FastStream, Aiogram, etc.)

## Package Structure

```
src/waku/
├── uow.py           # IUnitOfWork protocol (general infrastructure concern)
├── messaging/       # Messaging: IRequest, IEvent, Pipeline behaviors, MessageBus
│   ├── contracts/   # IMessage, IRequest, IEvent, IPipelineBehavior, HandlerType, MessageEnvelope, EnvelopeFactory
│   ├── context.py   # MessageContext (ContextVar-based, correlation/causation propagation)
│   ├── dispatcher.py # MessageDispatcher (handler/behavior resolution, no routing)
│   ├── endpoints/   # Endpoint model (ABC, LocalQueueEndpoint, EndpointEntry)
│   ├── handler.py   # MessageHandler, RequestHandler, EventHandler (unified hierarchy)
│   ├── handler_map.py # HandlerMap (unified message→handler registry)
│   ├── behaviors/   # Pipeline behaviors (TransactionalBehavior)
│   ├── pipeline/    # PipelineExecutor, behavior chain
│   ├── router.py    # MessageRouter, RoutingTable, route()/route_module() helpers
│   ├── transport/   # ITransport protocol (external transports)
│   ├── sqla/        # SQLAlchemy adapters (SqlAlchemyUnitOfWork)
│   ├── impl.py      # MessageBus (thin routing facade)
│   ├── interfaces.py # IMessageBus, ISender, IPublisher
│   └── modules.py   # MessagingModule, MessagingConfig, MessagingExtension, EndpointLifecycleExtension
├── di/              # DI helpers wrapping dishka (scoped, singleton, transient, etc.)
├── eventsourcing/   # Event sourcing extension
│   ├── contracts/   # Aggregate, Event envelope, Stream primitives
│   ├── decider/     # Decider pattern (functional event sourcing)
│   ├── projection/  # Read model projections (with SQLAlchemy adapter)
│   ├── serialization/ # Event serialization (adaptix-based)
│   ├── snapshot/    # Aggregate snapshots (with SQLAlchemy adapter)
│   ├── store/       # Event store (with SQLAlchemy adapter)
│   ├── upcasting/   # Event schema migration/upcasting
│   ├── handler.py   # EventSourcedCommandHandler
│   ├── modules.py   # EventSourcingModule, EventSourcingConfig
│   └── repository.py # EventSourcedRepository
├── extensions/      # Lifecycle hooks and extension registry
├── modules/         # Module system (@module decorator, DynamicModule, registry)
└── validation/      # Module validation rules
```

## Development Commands

### Environment Setup
- `task deps:install` - Install dependencies and pre-commit hooks
- `task deps:sync` - Sync project dependencies

### Code Quality
- `task lint` - Lint code with ruff (check + format check)
- `task format` - Format and fix code with ruff
- `task typecheck` - Type check with mypy, ty, and pyrefly
- `task check` - Run lint + typecheck
- `task pre-commit` - Run all pre-commit hooks

### Testing
- `task test` - Run tests with pytest
- `task test:cov` - Run tests with coverage (98% minimum)
- `task all` - Full check: lint, typecheck, spellcheck, tests with coverage

### Single File Operations (during development)
- `uv run ruff check path/to/file.py` - Lint specific file
- `uv run ruff check --fix path/to/file.py` - Fix specific file
- `uv run ruff format path/to/file.py` - Format specific file
- `uv run mypy path/to/file.py` - Type check specific file
- `uv run pytest path/to/test_file.py` - Run specific test

Use `uv run` for iterative development; use `task` commands before commits.

## Architecture

### Bootstrap Flow
`WakuFactory(RootModule).create()` → builds `ModuleRegistry` (topological sort) → builds dishka `AsyncContainer` → builds `ExtensionRegistry` → returns `WakuApplication`.

The app is used as an async context manager: `async with app, app.container() as c:` handles initialization (extension hooks), lifespan functions, and shutdown in order.

### Module System
Modules use `@module(providers=[], imports=[], exports=[], extensions=[])` decorator. Import/export boundaries enforce dependency management. Modules are topologically ordered for init (dependencies first) and shutdown (dependents first).

### Dependency Injection (dishka)
Provider helpers from `waku.di`: `singleton`, `scoped`, `transient`, `contextual`, `object_`, `many`, `provider`, `activator`. The `provided_type=` kwarg maps implementation to interface. Conditional activation via `when=Marker(...)` / `when=Has(Type)`.

### Messaging + Message Bus
- `IMessage` base, `IRequest[TResponse]` for commands/queries, `IEvent` for notifications (plain marker classes, not Protocols)
- `MessageHandler[TMessage, TResponse]` base → `RequestHandler[TRequest, TResponse]` and `EventHandler[TMessage]`
- `MessagingExtension.bind(msg_type, handler, *additional, behaviors=[...])` — unified registration
- `IPipelineBehavior` for cross-cutting concerns (logging, validation, transactions)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waku-py/waku](https://github.com/waku-py/waku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
