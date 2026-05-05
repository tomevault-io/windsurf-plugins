---
trigger: always_on
description: **Source**: [GitHub Spec-Kit: Spec-Driven Development](https://github.com/github/spec-kit/blob/main/spec-driven.md#the-nine-articles-of-development)
---

# AGENTS.md - Coding Agent Guidelines

---

## 🔒 Constitutional Foundation: Enforcing Architectural Discipline

**Source**: [GitHub Spec-Kit: Spec-Driven Development](https://github.com/github/spec-kit/blob/main/spec-driven.md#the-nine-articles-of-development)

**These rules are MANDATORY and take precedence over all other guidelines in this document.**

### Article I: Library-First Principle

**Rule**: Every feature in this project MUST begin its existence as a standalone library. No feature shall be implemented directly within application code without first being abstracted into a reusable library component.

**Requirements**:
- All new features must be designed as independent, reusable library components first
- Features cannot be implemented directly in application code
- Libraries must have clear boundaries and minimal coupling
- This ensures modular architecture from inception

**Example Violations**:
- ❌ Adding a new color calculation function directly in `index_files/fediverse.js`
- ❌ Implementing position algorithms inline in application code

**Correct Approach**:
- ✅ Creating `scripts/fediverse-processor/colors.go` as a library module
- ✅ Extracting reusable functions into separate modules/files first

### Article II: CLI Interface Mandate

**Rule**: All libraries must expose functionality through command-line interfaces.

**Requirements**:
- All CLI interfaces MUST:
  - Accept text as input (via stdin, arguments, or files)
  - Produce text as output (via stdout)
  - Support JSON format for structured data exchange
- Every capability must be accessible and verifiable through standard interfaces
- Prioritize observability and testability
- Prevent hiding functionality within opaque implementations

**Example Compliance**:
- ✅ `./fediverse-processor` accepts JSON input, produces JSON output
- ✅ `node scripts/fetch-fediverse-data.js --limit=100` (CLI arguments)
- ✅ Text-based logs and structured JSON output

### Article III: Test-First Imperative

**Rule**: All implementation MUST follow strict Test-Driven Development.

**Requirements**:
- No implementation code shall be written before:
  1. Unit tests are written
  2. Tests are validated and approved by the user
  3. Tests are confirmed to FAIL (Red phase)
- Generate comprehensive test suites FIRST
- Obtain user approval for tests
- Confirm tests fail (Red)
- THEN implement solutions to make tests pass (Green)
- This ensures behavior-driven design

**Workflow**:
1. 🔴 **RED**: Write failing tests first
2. ✅ **User Approval**: Get explicit approval for test suite
3. ✅ **Verify Failure**: Confirm tests fail as expected
4. 🟢 **GREEN**: Implement code to pass tests
5. ♻️ **REFACTOR**: Clean up while keeping tests green

**When to Apply**:
- All new features requiring code implementation
- Bug fixes (write test that reproduces bug first)
- Performance optimizations (write performance test first)

**Exceptions**:
- Documentation-only changes
- Configuration file updates
- Trivial refactoring with existing test coverage

---

## Project Overview

**fediverse-with-100k-stars** is a fork of Chrome Experiments' "100,000 Stars" - an interactive 3D WebGL visualization repurposed to display the Fediverse network. Built with Three.js 0.158.0 (ES modules), Mocha/Chai for testing, and vanilla JavaScript.

**Live original**: https://stars.chromeexperiments.com

---

## Build & Run Commands

### Development Server
```bash
# No build step required - static HTML/JS project
# Use any static file server:
python3 -m http.server 8000
# OR
npx serve .
# OR
npx http-server .
```

### Testing
```bash
# Unit tests (Mocha/Chai) - run in browser
# Start a local server and open tests/runner.html
python3 -m http.server 8000
# Then open http://localhost:8000/tests/runner.html

# Go tests for data processor
cd scripts/fediverse-processor && go test -v ./...
```

### Linting
```bash
# No linter configured - legacy codebase
```

---

## Project Structure

```
fediverse-with-100k-stars/
├── index.html              # Main entry point
├── AGENTS.md               # Agent guidelines (this file)
├── README.md               # Project documentation
├── data/                   # Generated data files (gitignored)
│   └── fediverse_final.json  # Processed Fediverse instance data (~17MB)
├── docs/                   # Documentation
│   ├── README.md           # Documentation index
│   ├── architecture/       # Architecture decision records
│   │   └── coordinate-systems.md
│   ├── plans/              # Implementation plans
│   │   ├── fediverse-implementation.md  # Main implementation plan
│   │   ├── dual-scene-architecture.md
│   │   └── ...
│   └── postmortems/        # Post-mortem analyses (15+ files)
│       └── README.md       # Postmortem index
├── scripts/                # Data processing scripts
│   └── fediverse-processor/  # Golang processor
│       ├── main.go         # CLI entry point
│       ├── cli.go          # Command-line interface
│       ├── colors.go       # Color calculation library
│       ├── positions.go    # Position calculation library
│       ├── types.go        # Data type definitions
│       ├── go.mod          # Go module definition

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r0k1s-i/fediverse-with-100k-stars](https://github.com/r0k1s-i/fediverse-with-100k-stars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
