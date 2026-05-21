---
trigger: always_on
description: **Aevatar Station** - Distributed AI agent management platform built on Microsoft Orleans and ABP Framework.
---

# CLAUDE.md

## System Overview

**Aevatar Station** - Distributed AI agent management platform built on Microsoft Orleans and ABP Framework.

### Architecture
- **framework/** - Orleans-based actor framework with event sourcing
- **station/** - Multi-tenant AI agent management platform
- **signalR/** - Real-time communication layer

**Tech Stack**: .NET 9, Orleans 9, MongoDB, Redis, ABP Framework, SignalR
**Patterns**: Event Sourcing, CQRS, Actor Model, Multi-tenancy

## Development Commands

```bash
# Build & Test
dotnet build
dotnet test
dotnet format && dotnet build --no-incremental  # Pre-commit

# Start Services (in order)
cd station/src/Aevatar.DbMigrator && dotnet run
cd ../Aevatar.AuthServer && dotnet run
cd ../Aevatar.Silo && dotnet run
cd ../Aevatar.HttpApi.Host && dotnet run

# Specific Tests
dotnet test framework/test/Aevatar.Core.Tests --filter "FullyQualifiedName~[TestClass]" --verbosity normal
```

## Core Concepts

### GAgent Framework
Distributed agents inherit from `GAgentBase<TState, TEvent>`:
- `[GAgent]` - Agent class marker
- `[EventHandler]` - Event handling methods
- `[LogConsistencyProvider]` - Event store config
- `[StorageProvider]` - State persistence config

### Event Flow
Client → SignalR Hub → Orleans Grains → Event Sourcing → State Persistence

## Technical Documentation References
- @~/framework/TECHNICAL_DOCUMENTATION.md - Framework architecture & patterns
- @~/station/TECHNICAL_DOCUMENTATION.md - Station platform implementation
- @~/framework/docs/MODULE_DOCUMENTATION.md - Module-specific details
- @source-control.md - Source control management strategies and workflows
- @technical-documentation-practices.md - Practices for creating and maintaining technical documentation
- @test-case-generation-guidelines.md - Guidelines for generating test cases when working on TDD

---

## Coding Directives

### Role
Expert C# developer specializing in scalable, maintainable backend systems.

### CRITICAL: Analysis-First Development
**STOP** - Before writing ANY code:
1. **Understand** - Fully comprehend the task requirements
2. **Analyze** - Break down using sequentialthinking and MECE principles  
3. **Research** - Find existing patterns with openmemory/grep/find
4. **Design** - Propose solution architecture and approach
5. **Confirm** - Present analysis and get approval BEFORE implementing

Example response pattern:
```
## Task Analysis
I understand you need [requirement summary].

## Breakdown (MECE)
1. Component A: [description]
2. Component B: [description]
3. Component C: [description]

## Existing Patterns Found
- Similar implementation in [file:line]
- Related pattern in [file:line]

## Proposed Solution
1. Create/modify [component]
2. Implement [pattern]
3. Test with [approach]

Shall I proceed with this approach?
```

### Core Principles
- **Maintainability over performance** - Clean, readable code is paramount
- **Minimal changes** - Smallest reasonable modifications to achieve goals
- **Consistency** - Match existing code style within files
- **SOLID principles** - Apply throughout implementation
- **Never break existing functionality** - Preserve working code

### Critical Rules
- ❌ **NEVER** use `--no-verify` when committing
- ❌ **NEVER** rewrite implementations without explicit permission
- ❌ **NEVER** remove comments unless provably false
- ❌ **NEVER** implement mock modes - use real data/APIs
- ❌ **NEVER** name things as 'improved', 'new', 'enhanced'
- ❌ **NEVER** start coding without analysis and proposed solution
- ❌ **NEVER** write implementation code before writing tests
- ❌ **NEVER** skip writing tests unless explicitly authorized
- ✅ **ALWAYS** analyze and propose BEFORE implementing
- ✅ **ALWAYS** write failing tests FIRST before any implementation
- ✅ **ALWAYS** ask for clarification vs assumptions
- ✅ **ALWAYS** add `ABOUTME:` comments to new files (2 lines)
- ✅ **ALWAYS** preserve unrelated working code
- ✅ **ALWAYS** follow strict TDD: Red → Green → Refactor cycle

### Implementation Standards
```csharp
// File header format (required for new files)
// ABOUTME: This file implements [core functionality]
// ABOUTME: [Brief description of purpose/responsibility]

// Agent pattern
[GAgent]
[StorageProvider(ProviderName = "PubSubStore")]
[LogConsistencyProvider(ProviderName = "LogStorage")]
public class MyAgent : GAgentBase<MyState, MyEvent>, IMyAgent
{
    [EventHandler]
    public async Task HandleEventAsync(SomeEvent @event)
    {
        // Update state, confirm events
        State.Property = @event.Data;
        await ConfirmEvents();
    }
}
```

## Testing Requirements

### STRICT TEST-DRIVEN DEVELOPMENT (TDD) - ABSOLUTELY MANDATORY

#### TDD is NOT OPTIONAL - It is a REQUIREMENT for ALL code changes

**The TDD Cycle MUST be followed for EVERY feature, bug fix, or code modification:**

1. **RED PHASE** - Write failing test FIRST
   - Define expected behavior through test
   - Run test to confirm it fails (proves test is valid)
   - NO implementation code allowed at this stage
   
2. **GREEN PHASE** - Write MINIMAL code to pass
   - Implement ONLY enough code to make test pass
   - No extra features or "nice to have" additions
   - Run test to confirm it passes
   

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aevatarAI/aevatar-station](https://github.com/aevatarAI/aevatar-station) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
