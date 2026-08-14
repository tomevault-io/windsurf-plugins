---
trigger: always_on
description: - When numbers are needed (performance, memory, file counts), run actual measurements locally
---

# TypeGuessr - Project Context

## Behavioral Rules

### Measure, Don't Estimate
- When numbers are needed (performance, memory, file counts), run actual measurements locally
- Prefer local execution over internet searches for empirical data
- Measurements without code_index are meaningless — code_index is required for all inference-related benchmarks

### Prioritization
- CI/test failures > fixing existing features > new features
- After context resumption, re-read brain.md and confirm the current task before starting work
- Only work on what the user requested. Do not start related tasks without asking

### Action Over Explanation
- When the user points out a problem, propose a fix immediately — the user already knows the problem
- Do not dismiss test failures as "pre-existing" without first investigating whether your changes could be the cause
- Propose fixing existing implementations before suggesting to scrap them
- Execute promised actions (todo.md updates, file additions) immediately — do not defer

### Context Management
- In long sessions, persist important intermediate results to files (they survive compaction)
- When presenting sub-agent results, assume the user has NOT read the raw output
- After context compaction, re-read brain.md and todo.md to recover state

### Critical Invariants
- NEVER pass code_index as nil — inheritance chains, duck typing, and type simplification all depend on it
- NEVER deploy performance-sensitive features without benchmarking on representative workloads first
- NEVER introduce concurrent access to shared data structures without considering thread-safety

## Project Overview

TypeGuessr is a Ruby LSP addon that provides **heuristic type inference** without requiring explicit type annotations. The goal is to achieve a "useful enough" development experience by prioritizing practical type hints over perfect accuracy.

**Core Approach:**
- Infers types from **method call patterns** (inspired by duck typing)
- Hooks into ruby-lsp's TypeInferrer to enhance Go to Definition and other features
- Uses variable naming conventions as hints
- Leverages RBS definitions when available
- Focuses on pragmatic developer experience rather than type correctness

**Key Example:**
```ruby
def fetch_comments(recipe)
  recipe.comments  # If 'comments' method exists only in Recipe class,
end                # infer recipe type as Recipe instance
```

**Key Information:**
- **Language:** Ruby 3.3.0+
- **Type:** Ruby LSP Addon (Gem)
- **Main Dependency:** ruby-lsp ~> 0.22
- **Author:** riseshia
- **Repository:** https://github.com/riseshia/type-guessr

## Development Workflow

### Setup
```bash
bin/setup
```

### Running Tests
```bash
bundle exec rspec
```

### Running Linter
```bash
bundle exec rubocop -a
```

### Running All Checks
```bash
bundle exec rspec && bundle exec rubocop -a
```

### Console
```bash
bin/console
```

### Testing Hover in Real LSP Environment
```bash
bin/hover-repl
```

REPL-style tool that spawns actual ruby-lsp server with TypeGuessr addon.
Waits for full project indexing (~20 seconds), then allows multiple hover queries:

```
> lib/type_guessr/core/config.rb 40 11
**Method Signature:** `() -> ?Hash[String, true | false]`
...
> exit
```

**Non-interactive mode** (for Claude Code debugging):
```bash
# Single query - outputs hover result and exits
bin/hover-repl lib/type_guessr/core/config.rb 40 11

# JSON output for programmatic use
bin/hover-repl lib/type_guessr/core/config.rb 40 11 --json
```

Use this to verify hover results match what users see in their editors.

### Verifying DSL Type Inference
```bash
bin/verify-inference sample/activerecord
bin/verify-inference sample/mongoid
bin/verify-inference sample/activerecord --verbose
```

Automated type inference verification against sample Rails/Mongoid projects.
Parses `test_inference.rb` lines with `# => ExpectedType` comments, starts
ruby-lsp with TypeGuessr addon, hovers over each method call, and checks
if the hover result contains the expected type string.

Output shows PASS/FAIL/SKIP for each line with a summary at the end.
Use `--verbose` to see TypeGuessr server logs.

**Adding new test cases:** Add a line to `test_inference.rb` with `# => Type`:
```ruby
user.name              # => String?
User.where(active: true) # => ActiveRecord::Relation[User]
```

The hover target is auto-detected as the last `.method_name` on the line.

## TDD Development Workflow

This project follows strict Test-Driven Development (TDD) practices.

### TDD Cycle: Red → Green → Refactor

1. **Red:** Write a failing test first
2. **Green:** Write minimal code to make the test pass
3. **Refactor:** Clean up code while keeping tests green
4. **Commit:** Only commit when all tests pass

## Important Conventions

1. **Language:** **ALL code-related content MUST be written in English:**
   - Commit messages, code comments, variable names, documentation
   - **Exception:** You may communicate with the user in Korean for clarifications

2. **Frozen String Literals:** All Ruby files use `# frozen_string_literal: true`

3. **Code Style:** Follows RuboCop rules defined in `.rubocop.yml`

4. **Testing:** Uses RSpec for testing

5. **Naming:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [riseshia/type-guessr](https://github.com/riseshia/type-guessr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
