---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**playwright-rust** is a Rust implementation of language bindings for Microsoft Playwright, following the same architecture as playwright-python, playwright-java, and playwright-dotnet.

### Vision and Design Philosophy

**Problem**: Rust developers need reliable, cross-browser testing, and Playwright is the modern standard for browser automation.

**Solution**: Provide production-quality Rust bindings for Microsoft Playwright that:
- Reuse Playwright's battle-tested server (don't reimplement browser protocols)
- Match Playwright's API across all languages (consistency)
- Leverage Rust's type safety and performance

**Key Principles:**
- **Microsoft-compatible architecture** - JSON-RPC to Playwright server (not direct protocol)
- **API consistency** - Match playwright-python/JS/Java exactly
- **Type safety** - Leverage Rust's type system for compile-time guarantees
- **Production quality** - Drive broad adoption
- **Testing-first** - Comprehensive test coverage from day one
- **Idiomatic Rust** - async/await, Result<T>, builder patterns

**Strategic Positioning:**
- **Not reinventing the wheel**: Uses official Playwright server for browser automation
- **Cross-language consistency**: Same API as Python/JS/Java/.NET implementations
- **t2t project driver**: Development driven by real-world need in [t2t repo](https://github.com/padamson/t2t) (browser testing for full stack app)

### Technology Stack

**Primary Language: Rust**
- **Why Rust**: Type safety, performance, modern async/await, great for developer tools
- **Async runtime**: tokio (de facto standard for async Rust)
- **Serialization**: serde + serde_json for JSON-RPC protocol
- **Process management**: tokio::process for Playwright server lifecycle

### Project Structure

```
playwright-rust/
├── crates/
│   └── playwright/                  # Single crate (consolidated from playwright-core in v0.7)
│       ├── src/
│       │   ├── api/                # Launch options, connect options
│       │   ├── protocol/          # Protocol objects (Page, Browser, Locator, etc.)
│       │   ├── server/            # Connection, transport, channel, object factory
│       │   ├── lib.rs             # Public exports
│       │   ├── assertions.rs      # Expect API (auto-retry assertions)
│       │   └── error.rs           # Error types
│       ├── tests/integration/     # Integration tests (632+ tests)
│       ├── examples/              # Usage examples
│       ├── fuzz/                  # Fuzz targets (cargo-fuzz)
│       └── Cargo.toml
├── drivers/                        # Playwright server binaries (gitignored)
├── supply-chain/                   # cargo-vet audit config
├── docs/
│   ├── implementation-plans/      # Gap analysis, version plans
│   ├── adr/                       # Architecture Decision Records
│   └── roadmap.md
└── README.md
```

## Development Approach

This project uses **test-driven development (TDD)** and **incremental delivery** with focus on Microsoft Playwright API compatibility.

### Specialized Development Agents

For complex workflows, use these specialized agents (located in `.claude/agents/`):

1. **TDD Feature Implementation Agent** (`tdd-feature-implementation.md`)
   - Use when: Implementing any new Playwright API feature
   - Automates: Red → Green → Refactor workflow, cross-browser testing, API compatibility checks
   - Ensures: Tests written first, API matches Playwright exactly, rustdoc complete

2. **Documentation Maintenance Agent** (`documentation-maintenance.md`)
   - Use when: Completing slices/versions, updating docs, releasing versions
   - Automates: Just-in-time doc updates, hierarchy enforcement, CHANGELOG generation
   - Ensures: README shows current features only, roadmap stays strategic, implementation plans stay current

3. **API Compatibility Validator Agent** (`api-compatibility-validator.md`)
   - Use when: Reviewing API implementations, validating compatibility
   - Automates: Cross-language API comparison, parameter validation, type checking
   - Ensures: API exactly matches playwright-python/JS/Java, no drift

4. **Release Preparation Agent** (`release-preparation.md`)
   - Use when: Preparing a version release (version bump, CHANGELOG, verification)
   - Automates: Pre-release checks, test verification, version management, validation
   - Ensures: All quality gates pass, CHANGELOG is complete, release process is smooth

#### Automatic Agent Invocation

**IMPORTANT**: Proactively use agents when user requests match these patterns:

**TDD Feature Implementation Agent** - Use automatically when user:
- Says "implement {feature}" or "add {method}"
- Mentions implementing a Playwright API (page.goto, browser.launch, etc.)
- Asks to "create a new feature" or "add support for"
- Example triggers: "implement page.screenshot()", "add browser.pdf() method"

**Documentation Maintenance Agent** - Use automatically when user:
- Says "Slice X complete", "Version Y done", or "finished Slice Z"
- Asks to "update documentation" or "update docs"
- Mentions "release" or "preparing for release"
- Says "version complete" or "slice finished"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [padamson/playwright-rust](https://github.com/padamson/playwright-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
