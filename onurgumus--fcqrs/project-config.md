---
trigger: always_on
description: FCQRS is an F# framework implementing Command Query Responsibility Segregation (CQRS) with Event Sourcing using Akka.NET actors.
---

# FCQRS Project Context for Claude

## Project Overview
FCQRS is an F# framework implementing Command Query Responsibility Segregation (CQRS) with Event Sourcing using Akka.NET actors. 

**Philosophy**: Use enterprise-grade distributed patterns for all applications (including CRUD) because the reliability and consistency guarantees are valuable from day one, and the framework handles the complexity overhead automatically.

## Architecture Components

### Core Modules
- **FCQRS** - Core framework with actor implementation
- **FCQRS.Model** - Domain modeling with validation and optics  
- **FCQRS.Serialization** - Custom serialization support
- **FCQRS.SQLProvider** - Database query provider
- **saga_sample/** - Saga pattern examples
- **sample/** - Basic CQRS examples

### Key Patterns

**Complete CQRS Flow:**
1. **Command Side**: Commands → Aggregates (cluster-sharded actors) → Events persisted
2. **Event Distribution**: Events flow to both read side and saga orchestration
3. **Read Side**: Events automatically update projections/read models 
4. **Query Side**: Optimized queries against read models
5. **Client Coordination**: CID-based subscriptions ensure clients know when read side is updated
6. **Side Effects**: Sagas handle external operations (emails, APIs) with retry/compensation

**Core Types:**
```fsharp
// Commands carry business intent
type Command<'CommandDetails> = {
    CommandDetails: 'CommandDetails
    CreationDate: DateTime
    Id: MessageId option
    Sender: ActorId option
    CorrelationId: CID
}

// Events represent what happened
type Event<'EventDetails> = {
    EventDetails: 'EventDetails
    CreationDate: DateTime
    Id: MessageId option
    Sender: ActorId option
    CorrelationId: CID
    Version: Version
}
```

### Domain Modeling
- **Validated types**: ShortString, LongString, CID, ActorId with ValueLens patterns
- **Predicate system**: For complex queries (Greater, Equal, And, Or, etc.)
- **Aether optics**: Functional lenses for nested data access
- **Validation framework**: Type-safe validation with detailed error reporting

### Actor System
- **Aggregates**: Business entities as cluster-sharded actors
- **Sagas**: Long-running processes, named as `originatorId__~Saga~_CID`
- **Event sourcing**: State rebuilt from events, snapshots every 30 events
- **Akka.NET integration**: Full clustering and distribution capabilities

## Key Files

### Model Layer (`/src/FCQRS.Model/`)
- `Model.fs` - Core domain types, validation, ValueLens patterns
- `Query.fs` - Query interface and DataEvent types

### Core Framework (`/src/FCQRS/`)
- `Actor.fs` - Aggregate actor implementations
- `Common.fs` - Shared utilities and types
- `Saga.fs` - Saga/process manager implementations

### Examples
- `sample/Command.fs` - Basic command handling example
- `saga_sample/` - Saga orchestration patterns

## Development Commands
```bash
# Build project
dotnet build

# Run samples  
dotnet run --project sample
dotnet run --project saga_sample
```

## Technical Stack
- .NET 9
- F# with functional programming patterns
- Akka.NET for actor system
- Custom JSON serialization
- Optional SQL providers for queries

## Architecture Principles
- **Event Sourcing**: Complete audit trail, state reconstruction
- **CQRS**: Separate read/write models
- **Actor Model**: Thread-safe, distributed processing
- **Functional**: Immutable data, validation, type safety
- **Domain-Driven**: Rich domain modeling with validation

## Critical Implementation Details

### Aggregate Development Pattern
1. **Create isolated functions**: `handleCommand` and `applyEvent` functions
2. **Wire with Akka.NET**: Use `init` and `initFactory` for actor system integration
3. **Cluster sharding**: Actors distributed across cluster nodes as virtual actors
4. **Garbage collection**: Actors can be passivated when inactive
5. **Thread safety**: Each actor processes messages sequentially

### Command/Event Flow with Saga Integration
1. **Command processing**: Sender subscribes to commands, waits for condition to yield true
2. **Event validation**: Events checked if they start a saga
3. **Saga startup sequence**:
   - Thread locked via `Ask` pattern
   - Event sent to saga starter
   - Saga persists initial event as `SagaStartingEvent`
   - Saga subscribes to mediator
   - Continue message sent to originator actor
   - Originator can continue processing
   - Event persisted and published
4. **Saga state management**: If event started saga, saga handles it again to switch states via `StateChangeEvent`

### Saga Architecture
- **Purpose**: Sagas take events and issue commands; Aggregates get commands and issue events
- **Naming convention**: `originator~~Saga~~correlationId`
- **Self-discovery**: Saga can find its originator from its name
- **Lifecycle**: Auto-remembered entities that auto-start
- **Correlation ID**: Critical for linking related messages and processes

### Key Concepts
- **Cluster-sharded actors**: Virtual actors that can be created anywhere in cluster
- **Passivation**: Automatic garbage collection of inactive actors
- **Thread safety**: Guaranteed through actor model's message-passing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OnurGumus/FCQRS](https://github.com/OnurGumus/FCQRS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
