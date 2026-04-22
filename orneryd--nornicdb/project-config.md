---
trigger: always_on
description: **Last Updated:** 2024-12-01
---

# NornicDB Development Guide for AI Agents

**Version:** 1.0.0  
**Last Updated:** 2024-12-01  
**Target Audience:** AI coding agents (Claude, GPT-4, etc.)

---

## Core Philosophy

NornicDB is an **enterprise-grade, high-performance graph database** that prioritizes:

1. **Neo4j Compatibility** - Drop-in replacement with zero code changes
2. **Performance** - 3-52x faster than Neo4j across benchmarks
3. **Correctness** - 90%+ test coverage, regression prevention mandatory
4. **Maintainability** - Clean architecture, separation of concerns, DRY principles
5. **Documentation** - Every public API fully documented with real-world examples

---

## 🎯 Golden Rules

### 1. **Prove Value Before Merging**

Every change must demonstrate **measurable improvements** without significant regressions:

- **Performance**: Benchmark before/after. Show ops/sec improvements.
- **Memory**: Profile memory usage. No >10% increases without justification.
- **Code Quality**: Maintain or improve test coverage (90% minimum).
- **Compatibility**: All Neo4j compatibility tests must pass.

**Example:**
```go
// ❌ BAD: No proof of improvement
func optimizeQuery() { /* new algorithm */ }

// ✅ GOOD: Benchmarked and documented
// BenchmarkQueryOptimization shows 2.3x speedup:
//   Before: 4,252 ops/sec
//   After:  9,780 ops/sec
//   Memory: -15% (reduced allocations)
func optimizeQuery() { /* new algorithm with proof */ }
```

### 2. **Test-Driven Bug Fixes**

**MANDATORY WORKFLOW** for all bugs:

1. **Write failing test** - Reproduce the exact bug condition
2. **Verify test fails** - Confirm it catches the bug
3. **Fix the bug** - Implement minimal fix
4. **Verify test passes** - Confirm fix works
5. **Add regression tests** - Prevent future occurrences

**Example from codebase:**
```go
// File: pkg/cypher/aggregation_bugs_test.go
// BUG #1: WHERE ... IS NOT NULL combined with WITH aggregation returns empty results

func TestBug_WhereIsNotNullWithAggregation(t *testing.T) {
    // 1. Setup test data that triggers bug
    setupAggregationTestData(t, store)
    
    // 2. Execute query that fails in production
    result, err := exec.Execute(ctx, `
        MATCH (f:File)
        WHERE f.extension IS NOT NULL
        WITH f.extension as ext, COUNT(f) as count
        RETURN ext, count
    `, nil)
    
    // 3. Assert expected behavior (this WILL fail before fix)
    require.NoError(t, err)
    assert.Equal(t, int64(2), extCounts[".ts"])
    assert.Equal(t, int64(3), extCounts[".md"])
}
```

See [.agents/testing-patterns.md](.agents/testing-patterns.md) for detailed testing guidelines.

### 3. **File Size Limits**

**Hard limit: 2500 lines per file**

When a file approaches 2500 lines:

1. **Identify separation of concerns** - What are the distinct responsibilities?
2. **Define clear contracts** - What interfaces connect the pieces?
3. **Extract cohesive modules** - Group related functionality
4. **Update imports** - Maintain clean dependency graph

**Example refactoring:**
```go
// Before: executor.go (3752 lines) ❌
// - Query parsing
// - Execution logic
// - Result formatting
// - Caching
// - Optimization

// After: Split into focused modules ✅
// executor.go (800 lines)      - Core execution orchestration
// parser.go (600 lines)         - Query parsing & AST
// optimizer.go (500 lines)      - Query optimization
// cache.go (400 lines)          - Result caching
// formatter.go (300 lines)      - Result formatting
```

See [.agents/refactoring-guidelines.md](.agents/refactoring-guidelines.md) for detailed strategies.

### 4. **Functional Go Patterns**

**Leverage function types for dependency injection:**

```go
// ✅ GOOD: Function interface enables runtime swapping
type EmbeddingFunc func(text string) ([]float32, error)

type SearchEngine struct {
    embedder EmbeddingFunc  // Inject different implementations
}

// Production: Use GPU-accelerated embeddings
engine := &SearchEngine{
    embedder: gpuEmbeddings.Generate,
}

// Testing: Use mock embeddings
testEngine := &SearchEngine{
    embedder: func(text string) ([]float32, error) {
        return []float32{0.1, 0.2, 0.3}, nil
    },
}

// Development: Use cached embeddings
devEngine := &SearchEngine{
    embedder: cachedEmbeddings.GetOrGenerate,
}
```

**Real example from codebase:**
```go
// pkg/storage/types.go - Storage interface for DI
type Engine interface {
    CreateNode(node *Node) error
    GetNode(id NodeID) (*Node, error)
    // ... more methods
}

// Multiple implementations:
// - MemoryEngine (testing)
// - BadgerEngine (production)
// - MockEngine (unit tests)
```

See [.agents/functional-patterns.md](.agents/functional-patterns.md) for advanced patterns.

### 5. **Documentation Standards**

**Every public API must have:**

1. **Package-level documentation** - What does this package do?
2. **Type documentation** - What does this type represent?
3. **Function documentation** - What does this do? When to use it?
4. **1-3 Real-world examples** - Show actual usage

**Example from codebase:**
```go
// Package cypher provides Neo4j-compatible Cypher query execution for NornicDB.
//
// This package implements a Cypher query parser and executor that supports
// the core Neo4j Cypher query language features. It enables NornicDB to be
// compatible with existing Neo4j applications and tools.
//

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orneryd/NornicDB](https://github.com/orneryd/NornicDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
