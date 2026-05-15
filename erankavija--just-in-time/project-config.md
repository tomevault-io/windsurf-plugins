---
trigger: always_on
description: This document provides guidelines for GitHub Copilot when working on the `just-in-time` codebase - a CLI-first issue tracking system designed for programmatic agents with focus on dependency graphs and quality gating.
---

# GitHub Copilot Instructions for Just-In-Time

This document provides guidelines for GitHub Copilot when working on the `just-in-time` codebase - a CLI-first issue tracking system designed for programmatic agents with focus on dependency graphs and quality gating.

## Project Overview

`just-in-time` (jit) is a repository-local issue tracker that implements:
- Explicit dependency graphs (DAG) between issues
- Optional and configurable issue typing and hierarchy based on labels
- Quality gating with automated and manual gates
- Machine-consumable plaintext storage (JSON files)
- Event logging for audit trail and observability
- (future) Coordinator-based agent orchestration

**Note:** We eat our own dog food - `just-in-time` is used to track issues in this very repository! 

## Core Design Principles

### 1. Functional Programming Paradigm

**Prefer functional programming patterns throughout the codebase:**
- Use immutability where practical; prefer returning new values over mutation
- Leverage iterators and functional combinators (map, filter, fold) over explicit loops
- Write pure functions without side effects whenever possible
- Use expression-oriented code rather than statement-oriented
- Employ higher-order functions and closures for abstraction
- Use `Result<T, E>` and `Option<T>` for error handling, avoiding panics in library code
- Use type-driven design with strong type safety

**⚠️ Pragmatic exceptions:**
- File I/O and system operations may require imperative code
- Coordinator daemon may use mutable state for tracking agent assignments
- Command-line interface (CLI) code can be more imperative where clarity benefits
- Always encapsulate imperative details behind clean, functional APIs

**Examples of preferred patterns:**
```rust
// GOOD: Functional style with iterator combinators
let blocked_issues = issues.iter()
    .filter(|i| i.is_blocked())
    .collect::<Vec<_>>();

// AVOID: Imperative loops when functional alternatives exist
let mut blocked_issues = Vec::new();
for issue in &issues {
    if issue.is_blocked() {
        blocked_issues.push(issue);
    }
}

// GOOD: Pure functions that return new values
fn compute_ready_issues(issues: &[Issue]) -> Vec<&Issue> { ... }

// GOOD: Expression-oriented code with Result
let issue = load_issue(&id)?;
let state = if issue.has_blocking_dependencies() {
    State::Blocked
} else {
    State::Ready
};
```

### 2. Test-Driven Development (TDD)

**Always follow TDD principles:**
1. Write tests first, implement minimal code to pass
2. Use property-based tests (`proptest`) for graph operations
3. Cover edge cases: empty deps, cycles, missing issues, concurrent claims
4. Verify invariants: DAG property, atomic operations, gate semantics

## Code Style and Conventions

### Safety and Correctness
- **No unsafe code**: The crate uses `#![deny(unsafe_code)]`
- **Validate all inputs**: Check preconditions and return `Result` with descriptive errors
- **Maintain invariants**: DAG property, atomic file operations, gate prerequisites
- **Comprehensive documentation**: All public APIs must have doc comments with examples
- **Clean code**: Run `cargo clippy` (zero warnings) and `cargo fmt` before committing
- **Push only valid states**: Fix all tests, warnings, and formatting issues immediately

### Naming Conventions
- Use clear, descriptive names that convey intent
- Method names should be verbs for actions (e.g., `add_dependency`, `claim_issue`)
- Predicates should start with `is_` or `has_` (e.g., `is_blocked`, `has_passing_gates`)
- Use conventional Rust naming: `snake_case` for functions/variables, `PascalCase` for types
- Use domain terminology: `issue`, `gate`, `dependency`, `assignee`, `coordinator`

### Error Handling
- Use `Result<T, Error>` for all fallible operations
- Define custom error types with `thiserror` for clear error messages
- Provide context in errors (e.g., "Issue 01ABC not found" not just "Not found")
- Avoid panics in library code; reserve `panic!` for unrecoverable internal bugs
- CLI layer can convert errors to user-friendly messages and exit codes

### Performance Considerations
- **Correctness first, then performance** - this is not a performance-critical system
- File I/O is the bottleneck; optimize JSON parsing if needed
- Avoid unnecessary allocations in hot paths (issue list queries)
- Use appropriate data structures (HashMap for ID lookups, Vec for lists)
- Profile before optimizing; measure actual bottlenecks
- Document performance characteristics for coordination operations

### Domain-Specific Guidelines
- **DAG enforcement**: All dependency operations must preserve acyclic property
- **Atomic operations**: File writes must be atomic (write temp + rename)
- **Event logging**: All state changes must be logged to `data/events.jsonl`
- **Gate semantics**: Issues cannot transition to `ready`/`done` with blocking gates
- **Assignee format**: `{type}:{identifier}` (e.g., "copilot:session-1")
- Use issue tracking and workflow terminology from Jira/GitHub Issues

## Repository Structure

- **/crates/jit**: Main crate implementing core functionality

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erankavija/just-in-time](https://github.com/erankavija/just-in-time) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
