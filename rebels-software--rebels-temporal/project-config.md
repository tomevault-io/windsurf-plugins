---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Rebels.Temporal** is a high-performance C# library for temporal event matching and correlation, designed for IoT, telemetry processing, and event-driven architectures. The library prioritizes:
- **Zero-allocation hot paths** for predictable, high-throughput performance
- **Allen's Interval Algebra** for temporal relations
- **Pure domain logic** with no external dependencies beyond .NET BCL
- **Flat namespace** (`Rebels.Temporal`) for simple API discovery

## Build and Test Commands

### Building
```bash
# Build entire solution
dotnet build

# Build in Release mode
dotnet build -c Release

# Build specific project
dotnet build src/Rebels.Temporal/Rebels.Temporal.csproj
```

### Testing
```bash
# Run all tests
dotnet test

# Run tests with filter (by namespace, class, or method name)
dotnet test --filter "MatchingTests"
dotnet test --filter "FullyQualifiedName~PointMatching"

# Run specific test method
dotnet test --filter "MethodName=Should_Match_Points_With_Tolerance"

# Run tests with detailed output
dotnet test -v detailed
```

### Benchmarks
```bash
# Run all benchmarks
cd benchmarks
dotnet run -c Release

# Run specific benchmark filter
dotnet run -c Release -- --filter *Sorted*
dotnet run -c Release -- --filter *PointMatching*
```

### Code Formatting
```bash
# Auto-format code following .NET standards
dotnet format
```

## Architecture

### Domain Model Philosophy

Rebels.Temporal is a **pure domain library**—it contains NO application layer, NO infrastructure layer, NO persistence, and NO external dependencies (ADR-4, ADR-7). The entire codebase exists in the domain layer.

### Core Concepts

The library defines four fundamental temporal concepts (see README.md domain model table):

1. **Temporal Event** (`ITemporalPoint`) - Point-in-time occurrences with a single timestamp
2. **Temporal Period/Interval** (`ITemporalInterval`) - Duration-based occurrences with Start and End
3. **Time Window** (`TimeWindow`) - Analytical time ranges for correlation (not domain occurrences)
4. **Temporal Relations** (`TemporalRelation`) - Allen's 13 interval algebra relations

### Fluent API Design

The primary API uses a fluent, zero-allocation interface accessed through `MatchTemporal`:

```csharp
// Pattern: MatchTemporal.<AnchorType>.With.<CandidateType>
MatchTemporal.Points.With.Points(...)      // Point → Point
MatchTemporal.Points.With.Intervals(...)   // Point → Interval
MatchTemporal.Intervals.With.Points(...)   // Interval → Point
MatchTemporal.Intervals.With.Intervals(...) // Interval → Interval (with Allen relations)
```

See `src/Rebels.Temporal/Matching/Execution/MatchTemporal.cs:21-170` for the fluent API structure.

### Visitor Pattern API (ADR-10)

Unlike typical .NET APIs, matching methods **do not allocate or return collections**. Callers provide a visitor implementing `IMatchVisitor<TAnchor, TCandidate>`:

```csharp
var buffer = new MatchPair<SensorReading, CommandEvent>[100];
var visitor = new BufferVisitor<SensorReading, CommandEvent>(buffer);

MatchTemporal.Points.With.Points(
    anchors, candidates, policy, ref visitor);

// Process results 0..visitor.MatchCount-1 in the buffer
// Also available: visitor.UnmatchedCount for observability
```

This enables zero-allocation hot paths (INV-3), gives callers full memory control, and provides built-in observability through `OnMatch` and `OnUnmatchedAnchor` callbacks.

### Algorithm Selection via InputOrdering

Performance depends critically on `InputOrdering` in `MatchPolicy`:

| InputOrdering | Algorithm | Complexity | When to Use |
|---------------|-----------|------------|-------------|
| `Both` | Dual-pointer scan | O(n+m) | Both collections sorted by timestamp (~255x faster) |
| `Candidates` | Binary search | O(n log m) | Only candidates sorted (e.g., from DB with ORDER BY) |
| `None` | Nested loops | O(n×m) | Unsorted data or very small datasets |

**Always prefer sorted data when possible.** The library validates declared ordering at runtime (INV-10).

### Project Structure

```
src/Rebels.Temporal/
├── Matching/
│   ├── Concepts/        # Core interfaces: ITemporalPoint, ITemporalInterval
│   ├── Execution/       # MatchTemporal (fluent API), IMatchVisitor, BufferVisitor, MatchPair
│   └── Policies/        # MatchPolicy, TimeTolerance, InputOrdering, AllowedRelations
tests/Rebels.Temporal.Tests/
├── Matching/            # Core matching tests
├── EdgeCases/           # Boundary conditions
└── Reference/           # Reference implementations
benchmarks/              # BenchmarkDotNet performance tests
docs/
├── adr/                 # Architecture Decision Records (16 ADRs)
└── invariants/          # Non-negotiable system rules (10 invariants)
```

**All public types live in a single namespace:** `Rebels.Temporal` (ADR-9, INV-4). Folder structure is for organization only.

## Critical Constraints (Invariants)

When writing code, these invariants **MUST NEVER be violated**:

### INV-1: Interval Start-End Constraint
All intervals must satisfy `Start <= End`. The library validates this at runtime.

### INV-2: DateTimeOffset Only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rebels-software/rebels-temporal](https://github.com/rebels-software/rebels-temporal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
