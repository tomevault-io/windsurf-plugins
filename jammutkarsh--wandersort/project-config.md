---
trigger: always_on
description: This document outlines the standard coding patterns, style, and rules that AI agents must follow when contributing to this codebase.
---

# Agent Coding Guidelines

This document outlines the standard coding patterns, style, and rules that AI agents must follow when contributing to this codebase.

## 1. Testing Strategy

- **Algorithm-focused Tests**: Unit tests must focus on algorithmic correctness, edge cases, concurrency constraints, memory limits, etc.
  - *Example:* Testing `walkRoot()` for correct traversal of a directory tree, including edge cases like symlinks, permission errors, and empty directories.
- **Skip Trivial Data Flows**: Avoid testing simple data parsing or standard read/write flows unless it contains complex business logic.
  - *Example:* Avoid writing tests for a simple HTTP handler that directly calls a database abstraction and returns JSON without any conditional logic.
- **Table-driven Tests**: Use [TableDrivenTestsarrays](https://go.dev/wiki/TableDrivenTests) to group related test cases of similar types into a single test function. Avoid creating multiple separate single-case test functions for similar logic.
  - *Example:*

    ```go
    func TestClassifyName(t *testing.T) {
        tests := []struct {
            name     string
            input    string
            expected string
        }{
            {"valid name", "john", "person"},
        }
        for _, tt := range tests {
            t.Run(tt.name, func(t *testing.T) { /* test logic */ })
        }
    }
    ```

- **Use Test Helpers**: Extract repetitive test setups into helper functions and use `t.Helper()` to ensure test failure line numbers remain accurate.
  - *Example:*

    ```go
    func setupTestDB(t *testing.T) *sql.DB {
        t.Helper()
        // setup and return db
    }
    ```

## 2. Documentation and Comments

- **Concise Language**: Code comments must be minimal, concise, and straightforward. Get straight to the point.
  - *Example:*
    `// Good: Computes the user's age in days.`
    `// Bad: This function takes a user object and calculates their age by subtracting...`
- **Technical Jargon**: Use relevant common technical jargon to minimize word count. Do not explain standard computer science terms (e.g., "hash function").
  - *Example:* Say "Computes the SHA-256 hash" instead of explaining what a hash function is.
- **References**: Add references or documentation links for uncommon terms which cannot be found on Wikipedia.
  - *Example:* `// Uses the Aho-Corasick algorithm (https://en.wikipedia.org/wiki/Aho-Corasick_algorithm).`
- **Public vs. Private Functions**:
  - **Public functions** require a docstring explaining what the function does, its parameters, and its return value. This rule only applies to functions performing complex computations; simple helpers (e.g., parsing a string to a positive int) do not need this.
    - **Constructors** *(an exception)*: Constructors like `New()` functions do not need a docstring because their purpose is obvious from context.
  - **Private functions** need a simple comment above the definition explaining their purpose. Private functions with self-documenting names and straightforward logic (< 5 lines) may omit comments.
  - *Example:*

    ```go
    // CalculateTrajectory computes the optimal path for the given mass and initial velocity.
    // It returns the coordinates and any error encountered during calculation.
    func CalculateTrajectory(mass, vel float64) ([]Coord, error) { ... }

    // No docstring needed for simple helper
    func parsePositiveInt(s string) int { ... }
    ```

- **API Documentation**: API handlers and endpoints must include Swagger doc-based comments (e.g., `// @Summary`, `// @Description`) so that Swagger documentation can be automatically generated via the CLI.
  - *Example:*

    ```go
    // HandleReset godoc
    // @Summary Reset all application data
    // @Description Deletes all scan sessions, file registry entries, content groups and group members in a single transaction. Irreversible.
    // @Tags Admin
    // @Produce json
    // @Success 200 {object} ResetResponse
    // @Router /internal/v1/admin/reset [post]
    func (h *Handler) HandleReset(c *gin.Context) { ... }
    ```

## 3. Naming Conventions

- **English-like Semantics**: Package and function names should combine to read like an English phrase.
  - *Example:* `location.Resolver()` is preferred as it clearly states "location resolver". Avoid `loc.GetRes()`.
- **Avoid Abbreviations**: Do not use abbreviations or acronyms in function or struct names unless they are commonly understood.
  - *Example:* Prefer `hashFile()` over `hf()`.
- **Descriptive Variables**: Variables and constants must have descriptive names that clarify their intent. Single-letter variables are only acceptable for common loop indices (`i`, `j`, `k`) or standard conventions (`err` for errors, `ctx` for context).
  - *Example:* `maxRetries` instead of `m`.
- **Generic Function Names**: Keep function names generic enough to allow underlying implementations to change.
  - *Example:* Use `hashFile()` instead of `BLAKE3Hash()`. Caller of `hashFile()` should not need to know the underlying hashing algorithm.

## 4. Code Structure & Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jammutkarsh/wandersort](https://github.com/jammutkarsh/wandersort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
