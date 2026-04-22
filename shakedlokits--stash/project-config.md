---
trigger: always_on
description: This document outlines the coding guidelines, development philosophy, and project structure established for the Apple Notes Sync Tool. These guidelines should be followed by any AI agent or developer working on this project.
---

# Agent Development Guidelines

This document outlines the coding guidelines, development philosophy, and project structure established for the Apple Notes Sync Tool. These guidelines should be followed by any AI agent or developer working on this project.

## Philosophy

We follow the **Zen of Python** as our guiding principle:

- **Beautiful is better than ugly**
- **Explicit is better than implicit**
- **Simple is better than complex**
- **Complex is better than complicated**
- **Flat is better than nested**
- **Sparse is better than dense**
- **Readability counts**
- **Special cases aren't special enough to break the rules**
- **Although practicality beats purity**
- **Errors should never pass silently**
- **Unless explicitly silenced**
- **In the face of ambiguity, refuse the temptation to guess**
- **There should be one-- and preferably only one --obvious way to do it**
- **Now is better than never**
- **Although never is often better than *right* now**
- **If the implementation is hard to explain, it's a bad idea**
- **If the implementation is easy to explain, it may be a good idea**

## Core Principles

### Simplicity
- Keep it simple. Don't over-engineer.
- Avoid premature optimization.
- Prefer one-line solutions when they're clearer (e.g., `pcregrep` over 40 lines of bash).
- Don't add features or complexity until they're needed.

### Explicitness
- Be explicit in naming, behavior, and user interactions.
- Use `read` commands for user prompts with clear messaging.
- Log before long-running operations (e.g., "Searching for note...").
- Error messages should be clear and actionable.

### Purity
- Utility functions should be pure: input → output, no side effects.
- Functions that perform I/O (file system, AppleScript) are integration functions.
- Separate concerns: parsing logic vs. I/O operations.

## Project Structure

### Directory Layout

```
src/
  lib/              # Pure and integration functions
  bashly.yml        # CLI configuration
  *_command.sh      # Command implementations

test/
  cases/                   # All test specs
    unit_*_spec.sh         # Unit tests (pure functions)
    integration_*_spec.sh  # Integration tests (I/O, AppleScript)
    e2e_*_spec.sh          # End-to-end tests (full CLI workflows)
  fixtures/                # Test fixture files (flat structure)
  approvals/               # Auto-generated approval files
  approvals.bash           # Approval testing framework
  approve                  # Test runner

dist/                # Generated CLI (from bashly)

Formula/
  stash.rb           # Homebrew formula (auto-updated on release)

.github/
  workflows/
    ci.yml           # CI pipeline (runs tests on push/PR)
    release.yml      # Release pipeline (build, publish, update formula)
```

### Naming Conventions

- **Utility functions**: `function_name.sh` (e.g., `strip_frontmatter.sh`)
- **Test specs**: `{type}_{function_name}_spec.sh` (e.g., `unit_strip_frontmatter_spec.sh`)
- **Approval files**: Custom names describing the test (e.g., `strip_frontmatter_with_apple_id`)

## Testing Approach

### Test Categories

1. **Unit Tests** (`unit_*_spec.sh`)
   - Test pure functions only
   - No I/O operations
   - No external dependencies (except standard tools like `pcregrep`)
   - Fast execution
   - Examples: `strip_frontmatter`, `get_id_from_frontmatter`, `update_frontmatter`

2. **Integration Tests** (`integration_*_spec.sh`)
   - Test I/O operations and external integrations
   - File system operations: read, write
   - AppleScript operations: create, read, update, delete notes
   - Require actual Apple Notes application
   - Include cleanup (create → test → delete)
   - Add WARNING comments at the top of the file

3. **E2E Tests** (`e2e_*_spec.sh`)
   - Test full CLI workflows
   - Mock integration functions by redefining them (not aliases)
   - Simulate bashly args with `declare -A args`
   - Test user interaction flows
   - **Must unset mocks after each test** to avoid pollution
   - Run with `SKIP_INTEGRATION=1` to skip integration tests
   - Examples: `push` command, `pull` command

### Approval Testing

We use `approvals.bash` for snapshot testing:

```bash
# Source approvals.bash and the function being tested
source approvals.bash
source "$LIB_PATH/strip_frontmatter.sh"

describe "strip_frontmatter"
  
  # Use custom approval file names for readability
  approve "cat $FIXTURES_PATH/with_apple_id.md | strip_frontmatter" \
    "strip_frontmatter_with_apple_id"
```

**Approval file naming**: Use descriptive names that indicate what's being tested:
- `strip_frontmatter_with_apple_id`
- `strip_frontmatter_no_frontmatter`
- `root_command_help`

**Environment variables** for clean paths:
- `$FIXTURES_PATH` - Path to test fixtures
- `$LIB_PATH` - Path to lib functions  
- `$CLI_PATH` - Path to CLI executable
- `$APPROVALS_BASH` - Path to approvals.bash (for portable sourcing)

### Test Execution

```bash
make test           # Run all tests (unit + integration + e2e)
make test-unit      # Run only unit tests (skips integration)
```

Environment variable:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shakedlokits/stash](https://github.com/shakedlokits/stash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
