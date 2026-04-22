---
trigger: always_on
description: A functional-style, persistence-friendly state machine library for .NET. Transitions return logical commands instead of performing side effects, enabling deterministic, testable, and replayable workflows. Designed primarily for actor-model architectures where state machines don't remain resident in memory.
---

# Copilot Instructions for Functional State Machine

## Project Overview

A functional-style, persistence-friendly state machine library for .NET. Transitions return logical commands instead of performing side effects, enabling deterministic, testable, and replayable workflows. Designed primarily for actor-model architectures where state machines don't remain resident in memory.

## Build & Test Commands

### Build the solution
```bash
dotnet build
```

### Run all tests
```bash
dotnet test
```

### Run tests for a specific project
```bash
# Core functionality tests
dotnet test FunctionalStateMachine.Core.Tests

# Command runner tests
dotnet test FunctionalStateMachine.CommandRunnerTests

# Diagram generation tests
dotnet test FunctionalStateMachine.Diagrams.Tests
```

### Run a single test file
```bash
dotnet test FunctionalStateMachine.Core.Tests --filter FullyQualifiedName~StateMachineFireTests
```

### Run with code coverage
```bash
dotnet test /p:CollectCoverage=true
```

## Architecture

### Core Design Pattern

The state machine uses a **fluent builder pattern** with four generic type parameters:

- **TState** - enum or class representing states
- **TTrigger** - abstract record hierarchy for trigger types
- **TData** - record type holding state-associated data
- **TCommand** - abstract record hierarchy for output commands

Example structure:
```csharp
public enum MyState { StateA, StateB }
public abstract record MyTrigger { public sealed record Activate : MyTrigger; }
public sealed record MyData(int Counter);
public abstract record MyCommand { public sealed record DoWork : MyCommand; }

var machine = StateMachine<MyState, MyTrigger, MyData, MyCommand>.Create()
    .StartWith(MyState.StateA)
    .For(MyState.StateA)
        .On<MyTrigger.Activate>()
            .TransitionTo(MyState.StateB)
            .Execute(() => new MyCommand.DoWork())
    .Build();
```

### Fire Execution Model

The `.Fire()` method is pure and stateless:
- **Input**: trigger, currentState, currentData
- **Output**: tuple of (newState, newData, commands)
- **Effect**: Returns a list of commands to be executed elsewhere; performs no side effects itself

### Key Architectural Layers

1. **FunctionalStateMachine.Core** - Core builder and state machine logic
2. **FunctionalStateMachine.Diagrams** - Mermaid diagram generation from machine definitions
3. **FunctionalStateMachine.CommandRunner** - DI integration for command dispatch
4. **FunctionalStateMachine.CommandRunner.Generator** - Source generator for command dispatchers
5. **VendingMachineSampleApp** - Interactive sample demonstrating command runners
6. **FunctionalStateMachine.Samples** - Example implementations (LightSwitch, Timer, ShoppingTrolley, SessionLogin)

## Key Conventions

### Test Organization

Tests are organized by feature in `FunctionalStateMachine.Core.Tests`:
- `StateMachineFireTests` - basic transition and data modification
- `StateMachineConditionalTests` - guard conditions and branches (`If`/`Else`/`Done`)
- `StateMachineHierarchyTests` - parent/child state relationships
- `StateMachineImmediateTransitionTests` - automatic transitions
- `StateMachineValidationTests` - fluent builder validation
- `StateMachineUnhandledTests` - unhandled trigger behavior

Use xUnit with minimal setup—tests are pure function tests requiring no mocks or DI.

### Fluent Builder Chaining

The builder uses a fluent pattern where `For(state)` returns a `StateConfiguration` that supports:
- `.On<TTrigger>()` - defines trigger handler for specific trigger type
- `.OnEntry()` - entry commands when entering state (multiple overloads for different signatures)
- `.OnExit()` - exit commands when leaving state
- `.TransitionTo()` - target state for transition
- `.ModifyData()` - functional update to state data
- `.Execute()` / `.ExecuteSteps()` - return command(s)
- `.If()` / `.ElseIf()` / `.Else()` / `.Done()` - conditional branches

### Record-Based Types

The library heavily uses C# records:
- **States**: typically enums, but can be records
- **Triggers, Data, Commands**: always records
- Trigger and Command types use sealed record inheritance for polymorphic dispatch
- `.ModifyData()` uses `with` expressions for immutable updates

### Data Binding in Transitions

Data updates flow through the builder:
```csharp
.On<MyTrigger.Increment>()
    .ModifyData(data => data with { Count = data.Count + 1 })
    .Execute(/* receives updated data context if needed */)
```

### Validation

The `.Build()` method validates:
- Initial state is set
- All referenced states are defined
- Trigger handlers are properly configured

Validation errors throw `InvalidOperationException`.

### Hierarchical States

Parent/child relationships are defined via builder:
```csharp
.For(ParentState)
    .WithInitialSubState(ChildState)
    // configuration...
```

Child states can have their own transitions; unhandled triggers propagate to parent.

### Immediate Transitions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leeoades/FunctionalStateMachine](https://github.com/leeoades/FunctionalStateMachine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
