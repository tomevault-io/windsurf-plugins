---
trigger: always_on
description: Read .cursor/rules/spec.mdc
---

# LLML Project - Claude AI Documentation

## Core LLML Specification

Read .cursor/rules/spec.mdc

## Coding Rules

Reference .cursor/rules/{language = ["py", "ts", "go", "rs"]}.mdc

## Project Overview

LLML (Lightweight Language Markup Language) is **React for Prompts** - a multi-language compositional primitive that revolutionizes AI context engineering. Just as React transformed web development by making complex UIs composable and maintainable, LLML transforms AI development by making complex contexts composable and maintainable.

The project provides identical functionality across four programming languages: Python, TypeScript/JavaScript, Rust, and Go, converting nested data structures into human-readable, XML-like markup optimized for AI model attention.

### Core Purpose
- **Compositional Context Engineering**: Build complex AI contexts from simple, reusable components
- **Declarative AI Interactions**: Describe what your prompt should contain, not how to format it
- **Maintainable AI Systems**: Replace brittle string concatenation with robust data composition
- **Structured Document Creation**: Generate documents with clear hierarchy from data structures
- **Data Serialization**: Convert data into more readable format than JSON/YAML for AI applications

### Key Benefits
- **Component-Like Composition**: Build complex prompts from simple, reusable pieces
- **Declarative Approach**: Focus on what you want, not how to format it
- **Maintainable & Robust**: Changes to data automatically propagate without breaking
- **Zero Configuration**: Works out-of-the-box with sensible defaults
- **Consistent Output**: Identical results across all language implementations
- **LLM-Optimized**: Structured format reduces AI model cognitive load and improves performance
- **Developer Experience**: Type-safe, predictable, debuggable context engineering
- **Extensible Formatters**: Customizable formatter system like React's component system

## Project Structure

```
/Users/knrz/Git/zenbase-ai/llml/
├── README.md                 # Main project documentation
├── justfile                  # Cross-language task runner
├── py/                       # Python implementation
│   ├── README.md
│   ├── pyproject.toml
│   ├── src/
│   │   └── zenbase_llml/
│   │       ├── llml.py      # Main implementation
│   │       └── formatters/  # Formatter system
│   │           ├── base/    # Base type formatters
│   │           └── vibe_xml/ # VibeXML formatters
│   └── tests/
├── ts/                       # TypeScript implementation
│   ├── README.md
│   ├── package.json
│   ├── src/
│   │   ├── index.ts         # Main implementation
│   │   └── formatters/      # Formatter system
│   │       ├── base/        # Base type formatters
│   │       └── vibe-xml/    # VibeXML formatters
│   └── tests/
├── rs/                       # Rust implementation
│   ├── README.md
│   ├── Cargo.toml
│   ├── src/
│   │   ├── lib.rs           # Main implementation
│   │   └── formatters.rs    # Core formatting logic
│   └── tests/
└── go/                       # Go implementation
    ├── README.md
    ├── go.mod
    ├── pkg/llml/
    │   └── llml.go          # Main implementation
    └── tests/
```

Each language implementation is completely self-contained with its own:
- Source code and utilities
- Dependency management files
- Comprehensive test suites
- Language-specific documentation

## Development Environment

### Task Runner: Just
The project uses `just` as a cross-platform task runner for coordinating commands across all implementations.

#### Key Commands
```bash
# Run tests for all languages
just test

# Language-specific commands
just py <command>    # Run command in Python environment (uv)
just ts <command>    # Run command in TypeScript environment (bun)
just go <command>    # Run command in Go environment (go)
just rs <command>    # Run command in Rust environment (cargo)

# AI commands
just claude
just claude -p "your prompt here"      # Run Claude Code CLI
just gemini
just gemini -a -p "your prompt here"   # Run Gemini CLI
```

### CLI Tools

- Instead of `find`, use `fd`
- Instead of `grep`, use `rg`
- Instead of `sed`, use `rg+(awk|sed)` or ast-grep: `ast-grep -p '$A && $A()' -r '$A?.()'`

### Development Tools by Language
- **Python**: `uv` for dependency management, `pytest` for testing, `ruff` for linting
- **TypeScript**: `bun` for runtime and package management, `vitest` for testing
- **Rust**: `cargo` for dependency management and testing
- **Go**: Standard Go toolchain (`go test`, `go mod`)

### Configuration Options
- **Custom Formatters**: Extensible formatter system for specialized data types (like React components)
- **Formatter Composition**: Combine and override default formatters for custom behavior
- **Type-Specific Processing**: Define custom formatting logic for any data type
- **Component-Like Reusability**: Build libraries of reusable prompt components

For a deep dive into the React analogy and compositional patterns, see [REACT.md](REACT.md).

## Testing & Quality Assurance

### Cross-Language Consistency
The test suites ensure that all four implementations produce identical output for the same input data, maintaining consistency across the entire project.

## Usage Examples

### 1. Compositional Context Engineering
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zenbase-ai/llml](https://github.com/zenbase-ai/llml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
