---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

OpenSleigh is a distributed saga management library for .NET that enables reliable, event-driven orchestration across microservices. It implements the saga pattern with choreography using an outbox pattern for guaranteed message delivery.

**Key Facts:**
- **Language:** C# 12 with nullable reference types enabled
- **Target Frameworks:** net8.0 and net9.0 (multi-targeting)
- **Main Solution:** `src/OpenSleigh.sln`
- **Primary Branch:** `develop` (use this for PRs, not main/master)
- **License:** Apache 2.0
- **Current Version:** 3.0.6 (defined in `Versions.props`)

## Build and Test Commands

**CRITICAL: Always work from the `src/` directory for build operations.**

### Standard Development Workflow

```bash
cd src/

# 1. Restore dependencies (ALWAYS run first to avoid package resolution issues)
dotnet restore

# 2. Build the solution
dotnet build -c Release        # Expect ~174 nullable warnings (acceptable), 0 errors required

# 3. Run unit tests (~105 tests, no infrastructure needed, ~5-10 seconds)
dotnet test --framework net9.0 --filter "Category!=E2E&Category!=Integration"

# 4. Run a specific test class or method
dotnet test --framework net9.0 --filter "FullyQualifiedName~SagaRunnerTests"

# 5. Verify code formatting
dotnet format --verify-no-changes

# 6. Create NuGet packages (outputs to repo root packages/ directory)
dotnet pack -c Release
```

### Integration Tests

Integration tests require Docker infrastructure (MongoDB, RabbitMQ, Kafka, SQL Server, PostgreSQL):

```bash
# From repo root, start infrastructure
cd tests/infra
docker-compose up -d           # Wait 10-15 seconds for services to initialize

# Return to src/ and run integration tests (~20-30 seconds)
cd ../../src
dotnet test --framework net9.0 --filter "FullyQualifiedName!~Cosmos&Category=Integration"

# If tests fail, clean up and restart infrastructure
cd ../tests/infra
docker-compose down && docker-compose up -d
```

**Docker credentials** (for debugging connection issues):
- SQL Server: `SA_PASSWORD=Sup3r_p4ssword123`
- PostgreSQL: `POSTGRES_PASSWORD=Sup3r_p4ssword123`
- RabbitMQ vhost: `/opensleigh-tests`

**Note:** Cosmos tests are excluded (not fully implemented). E2E tests are disabled in CI.

## High-Level Architecture

### Core Concepts

**Saga:** A long-running, distributed transaction coordinated through messages. Each saga instance maintains state and reacts to messages by executing handlers and publishing new messages.

**Outbox Pattern:** All messages published by sagas are stored in an outbox table/collection and processed asynchronously by a background service. This ensures at-least-once delivery with exactly-once processing guarantees.

**Correlation:** Each saga flow is identified by a `CorrelationId` that flows through all related messages, enabling distributed tracing and state management.

### Three-Layer Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                         │
│  - Saga implementations (ISaga<TState>)                      │
│  - Message handlers (IHandleMessage<TMessage>)               │
│  - Message definitions (IMessage)                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    OpenSleigh Core                           │
│  - Saga orchestration (SagaRunner, SagaExecutionService)     │
│  - Message processing pipeline (MessageProcessor)            │
│  - Outbox management (OutboxBackgroundService)               │
│  - Instance lifecycle (SagaInstance, SagaInstanceFactory)    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────┬────────────────────────────────┐
│   Persistence Layer         │     Transport Layer            │
│  - ISagaStateRepository     │  - IMessageSubscriber          │
│  - IOutboxRepository        │  - Message routing             │
│  - Implementations:         │  - Implementations:            │
│    • Mongo                  │    • RabbitMQ                  │
│    • PostgreSQL             │    • Kafka                     │
│    • SQL Server             │                                │
│    • InMemory               │                                │
└────────────────────────────┴────────────────────────────────┘
```

### Key Abstractions and Their Locations

**Saga Definition:**
- `ISaga` / `ISaga<TState>`: Marker interface for sagas (src/OpenSleigh/ISaga.cs)
- `Saga<TState>`: Base class providing `Publish<TMessage>()` method (src/OpenSleigh/Saga.cs)
- `IStartedBy<TMessage>`: Marks the message that initiates a saga (src/OpenSleigh/Transport/IStartedBy.cs)
- `IHandleMessage<TMessage>`: Interface for handling messages (src/OpenSleigh/Transport/IHandleMessage.cs)

**Saga Lifecycle:**
- `ISagaInstance` / `SagaInstance<TState>`: Execution context with state, outbox, correlation tracking, and lock management (src/OpenSleigh/ISagaInstance.cs, SagaInstance.cs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mizrael/OpenSleigh](https://github.com/mizrael/OpenSleigh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
