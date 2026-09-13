---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

# CLAUDE.md

This file provides guidance when working with code in this repository.

## DO NOT EVER USE ALLOWS WITHOUT EXPLICIT USER PERMISSION. EVER.

This is the most important rule.

## DO NOT SIMPLIFY, RELAX, OR DELETE TESTS. EVER.
## Error-Focused testing is always the strategy.

Tests are meant to CATCH errors. If you delete them, simplify, relax, or ANY OTHER VARIATION of reducing test strictness,
you remove the point of having tests in the first place. You want tests to error, that means you found a bug. Instead,
analyze it, figure out the root cause, and fix it. You can modify the tests if they are incorrect, but do not simply
delete or simplify them to get the error to go away. This is your number one priority directive, always, no matter what.

## Project Overview

**Merlin** is an intelligent AI coding assistant with multi-model routing, automatic task decomposition, and comprehensive validation.

**THIS IS NOT A PRODUCTION OR PUBLIC PROJECT:**
- No backward compatibility requirements
- Remove code instead of deprecating
- Breaking changes are acceptable and encouraged
- Focus on cleanliness over compatibility

**Core Features:**
- Multi-tier model routing with automatic escalation
- **Recursive Step-Based Execution** (see EXECUTION_MODEL.md)
  - Agent returns `String | TaskList`
  - Full tool access at all times
  - Exit requirements validate step completion
  - Hard/soft error classification with retry logic
- Parallel task execution with conflict detection
- Multi-stage validation pipeline
- Terminal UI with real-time progress monitoring
- TypeScript runtime for agent code execution

**Your workflow**
- Assess necessary changes and current implementation from module docs
- Make the changes
- Update module docs and any other relevant documentation with your changes
- Add/verify testing covers those changes, preferrably with fixtures
- Run verify.sh to confirm
- Do not make new documentation files unless explicitly asked to

## Architecture

**Cargo workspace with these main crates:**

- `merlin-core` - Fundamental types, traits, error handling
- `merlin-context` - Context management, file indexing
- `merlin-languages` - Language backends (Rust via rust-analyzer)
- `merlin-providers` - External API providers (Groq, OpenRouter, Anthropic)
- `merlin-local` - Local model integration via Ollama
- `merlin-routing` - Task analysis, model tier selection, metrics
- `merlin-agent` - Agent execution, validation, orchestration
- `merlin-cli` - Command-line interface and TUI
- `merlin-tooling` - TypeScript runtime, file operations, bash execution
- `integration-tests` - Fixture-based integration testing

**Model Routing Flow:**
1. Analyze task complexity and intent
2. Select appropriate model tier
3. Execute with tool registry
4. Validate results
5. Escalate to higher tier on failure (up to 3 retries)

**Task Execution:**
- Agent decides: Return string OR decompose into TaskList
- Recursive decomposition: Steps can themselves return TaskLists
- Exit requirements: Each step validates completion
  - Callback validators: `file_exists`, `file_contains`, `command_succeeds`, etc.
  - Pattern matching: Regex validation
  - Named validators: Integration with ValidationPipeline
- Context specification per step
  - File patterns (glob)
  - Previous step results
  - Explicit content injection
- Retry logic:
  - Hard errors → escalate model tier
  - Soft errors → retry with feedback
  - Max 3 attempts per step
- Transactional file operations with rollback support
- Conflict detection prevents concurrent file modifications

## Repository Rules

### Strict Linting

**Extremely strict clippy configuration:**
- ALL clippy lints denied (all categories: pedantic, nursery, restriction, etc.)
- `missing_docs` denied - all public items must have doc comments
- No panic macros: `.unwrap()`, `.expect()`, `todo!()`, `unimplemented!()`, `unreachable!()`
- No `println!()`/`eprintln!()` - use `tracing` macros instead
- All code uses `Result<T, E>` with proper error handling

**Critical requirements:**
- **NEVER add `#[allow]` or `#[cfg_attr(test, allow(...))]` without EXPLICIT user permission**
  - If clippy complains, FIX THE CODE, do not silence the warning
  - This applies even for "trivial" lints like `min_ident_chars` or `excessive_nesting`
  - Refactor code to satisfy clippy's requirements
  - The ONLY exception is if the user explicitly says "add an allow for X"
- Do NOT use `cargo clean` - on ICE, delete only `target/{debug,release}/incremental/`, not entire profile
- Must pass `./scripts/verify.sh` with zero errors before completion

### Code Quality

**Rust Edition 2024:**
- Prefer RPITIT (Return Position Impl Trait In Trait) over `async-trait`
- Leverage gen blocks and async generators

**Documentation:**
All public items need doc comments with:
- Brief one-line summary
- Detailed explanation for complex items
- `# Errors` section for `Result` returns
- `# Examples` when helpful

### Repository Maintenance

**When modifying this project:**
1. Keep CLAUDE.md updated with behavioral changes and new patterns
2. Do NOT add implementation comments ("Phase 3 implementation", "Deleted function here", etc.)
3. Update CLAUDE.md if adding new repository rules or critical constraints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BigBadE/agentic_optimizer](https://github.com/BigBadE/agentic_optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
