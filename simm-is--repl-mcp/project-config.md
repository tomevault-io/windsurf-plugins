---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This MCP server is built on [mcp-toolkit](https://github.com/metosin/mcp-toolkit) providing a clean, simplified architecture with minimal dependencies. All 51 tools follow consistent patterns for reliability and ease of use.

## Instructions

1. **Interactive Development**: Use the MCP tools for Clojure development in preference to default text editing. Start with `eval` to test code interactively before committing changes.
2. **Code Quality**: Use `lint-code` for immediate feedback during development, `lint-project` for comprehensive checks, and `setup-clj-kondo` to initialize linting in new projects.
3. **File Management**: When adding new code, use the standard `Edit` tool followed by `load-file` to reload changes into the REPL. This maintains REPL state while updating code.
4. **Refactoring**: Use structural editing tools for complex refactoring to maintain code integrity. Fall back to `Edit` + `load-file` if issues arise.
5. **Testing**: Use `create-test-skeleton` for comprehensive test generation, `test-all` for full test runs, and `test-var-query` for targeted testing.
6. **Debugging**: Apply direct fixes for obvious issues. For complex problems, use systematic investigation with appropriate tools.
7. **Dependencies**: Use `add-libs` for runtime dependency addition (Clojure 1.12+), `sync-deps` after updating deps.edn, and `check-namespace` to verify availability.

### Workflow Philosophy

**Simple, Direct, Effective**: The mcp-toolkit foundation enables straightforward tool usage with consistent patterns and reliable error handling.

**Tool Selection Principles:**
- Use the most direct tool for the task
- All tools follow the same pattern: input → processing → structured output
- Error handling is built-in - tools return clear error messages
- No complex state management - each tool call is independent

### Tool Categories

**Evaluation (2 tools)**
- `eval` - Execute code in REPL with proper namespace context
- `load-file` - Load files to update REPL state

**Refactoring (11 tools)**
- `clean-ns` - Organize imports and namespaces
- `rename-function-*` - Rename across files or projects
- `extract-function` - Extract code into new functions
- `find-symbol` - Locate symbol definitions and usages

**Navigation & Analysis (14 tools)**
- `call-hierarchy` - Trace function relationships
- `usage-finder` - Find all usages with context
- `ns-vars`, `ns-list` - Explore namespaces
- `info`, `eldoc` - Get documentation

**Testing (3 tools)**
- `create-test-skeleton` - Generate test templates
- `test-all` - Run entire test suite
- `test-var-query` - Run specific tests

**Structural Editing (10 tools)**
- Session-based editing with zipper navigation
- Safe code transformations preserving structure

**Performance (2 tools)**
- `profile-cpu` - CPU usage analysis with flamegraphs
- `profile-alloc` - Memory allocation profiling

**Code Quality & Analysis (7 tools)**
- `lint-code` - Check code strings
- `lint-project` - Analyze entire codebases
- `setup-clj-kondo` - Initialize linting
- `analyze-project` - Get full AST analysis data
- `find-unused-vars` - Find unused variables and functions
- `find-var-definitions` - Find variable/function definitions
- `find-var-usages` - Find all variable/function usages

**Dependencies (3 tools)**
- `add-libs` - Hot-load dependencies
- `sync-deps` - Sync from deps.edn
- `check-namespace` - Verify availability

### Code Quality & Analysis Workflow

**Integrated Analysis**: clj-kondo provides comprehensive AST-based code analysis with zero configuration required.

**Quick Start:**
1. Run `setup-clj-kondo` once per project to initialize
2. Use `lint-code` during development for immediate feedback  
3. Run `lint-project` before commits for comprehensive checks
4. Use analysis tools for codebase understanding and refactoring

**Key Features:**
- Automatic config detection from `.clj-kondo/config.edn`
- Library-specific rules via `copy-configs: true`
- Clear error/warning distinction
- Fast performance even on large codebases
- Full AST analysis for vars, usages, and definitions

**Analysis Tools Workflow:**
```clojure
;; Find all unused vars for cleanup
find-unused-vars:
  paths: ["src"]

;; Get comprehensive project analysis
analyze-project:
  paths: ["src" "test"]

;; Find specific variable definitions
find-var-definitions:
  paths: ["src"]
  name-filter: "my-function"

;; Find all usages of a namespace
find-var-usages:
  paths: ["src"]
  namespace-filter: "myapp.core"
```

**Use Cases:**
- **Cleanup**: Find unused functions and variables
- **Refactoring**: Understand variable usages before changes  
- **Architecture**: Analyze dependencies and relationships
- **Onboarding**: Explore codebase structure systematically

### Dependency Management

**Hot-loading Support**: Add dependencies without restarting your REPL (Clojure 1.12+).

**Simple Workflow:**
```clojure
;; Check if library is available
check-namespace: "hiccup.core"

;; Add new dependency
add-libs: {hiccup/hiccup {:mvn/version "1.0.5"}}

;; Sync after editing deps.edn
sync-deps
```

**Requirements:**
- Clojure 1.12+ for `add-libs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simm-is/repl-mcp](https://github.com/simm-is/repl-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
