---
trigger: always_on
description: > **Related documents**:
---

# GitHub Copilot Instructions

> **Related documents**: 
> - Project Structure: project-structure.md in .github - Technical architecture and design decisions
> - Development Roadmap: roadmap.md in .github - TODO, plans, and completed work
> - C Port Notes: c_port.md in .github - C implementation details, API constraints, and design decisions

**IMPORTANT for AI agents**: When documenting challenging coverage cases, complex algorithms, or significant architectural decisions, update `project-structure.md` (not this file). This instructions file focuses on workflows and conventions.

This repository is a Lua implementation of the Ariadne diagnostics renderer. Originally ported from the Rust `ariadne` library, it now serves as the primary implementation with architectural improvements for performance and clarity.

## Quick Context

- **Language**: Lua 5.1+ with UTF-8 support (tested against Lua 5.1/LuaJIT)
- **Dependencies**: `lua-utf8`, `luaunit`, optional `luacov`
- **Tests**: Run `lua test.lua` from project root
- **Coverage**: 100% (all reachable code covered, 55 tests passing in stable state)
- **Key files**: 
  - `ariadne.lua` - All runtime code (~1500 lines)
  - `test.lua` - Exhaustive regression suite (~1400 lines)

**Current development state**: Check `roadmap.md` for active feature work. During feature development, tests may temporarily fail - this is expected. Always check roadmap to understand which tests are work-in-progress vs. regressions.

For detailed architecture, see project-structure.md.

For development status and plans, see roadmap.md.

---

## Collaboration Workflow (for AI agents)

When implementing new features (like line width limiting):

### Test-Driven Development
1. **Agent writes test cases first** (following existing test patterns in `test.lua`)
2. **User reviews and approves** test cases for correctness and completeness
3. **Tests define expected behavior** before any implementation

### Implementation Guidance

⚠️ **CRITICAL: Think Before Providing Solutions**

**MANDATORY workflow before providing ANY implementation suggestion**:
1. **Think deeply** about the requirement (understand what user really needs)
2. **Think again** about potential issues (edge cases, performance, correctness)
3. **Think third time** about the approach (is this the best solution?)
4. **Only then** provide the suggestion (pseudo-code or algorithmic approach)

**User's explicit requirement**: "Don't rush to write code. Think more. Think at least three times and make sure there are no issues with the approach before providing it. Only write code when I explicitly request it."

- **Never write code immediately** - always think first
- **Never provide code unless explicitly requested** - give approach/algorithm instead
- **Self-check rigorously** - verify your thinking before presenting
- **Ask questions** when uncertain - don't guess or assume

**When providing implementation suggestions**:
1. Identify which functions need modification
2. Provide pseudo-code or algorithmic approach (NOT full implementation)
3. Highlight key technical points (UTF-8 handling, edge cases, etc.)
4. Point out open questions that need user confirmation

**After agent provides suggestions**:
1. User evaluates feasibility and makes final architectural decisions
2. User writes actual code implementation
3. Agent assists with debugging and analysis when problems arise

### Iterative Development
- Start with simple cases, add complexity gradually
- Run tests frequently: `lua test.lua` after each change
- Agent helps identify edge cases and potential issues
- Both parties discuss trade-offs and design decisions

### Code Ownership
- **User maintains full control** over code quality and architecture
- **Agent acts as pair programmer**: review, suggest, assist (not implement)
- **User makes all final decisions** on implementation details
- Agent never writes production code without explicit user request

---

## Agent Guidelines

### Local Conventions and Patterns

- Use ASCII glyphs in tests to keep expectations stable unless intentionally testing Unicode
- Prefer pure functions and local helpers; avoid capturing globals other than standard Lua libraries
- When adding API surface, expose it via `ariadne.lua` and add matching tests in `test.lua`

### Test Development Workflow

1. Run tests: `lua test.lua`
2. Collect coverage: `rm -f luacov.* && lua -lluacov test.lua && luacov ariadne.lua > /dev/null`
3. Find uncovered lines: `rg -C 3 '^\*+0 ' luacov.report.out`
   - The luacov.report.out format: `<hit_count> <code>` where `***0` (variable-length stars) means uncovered
   - Use context to identify which function/section contains uncovered code
4. Add targeted tests for uncovered branches
5. Verify coverage again

### Test Writing Patterns

- **All tests must verify complete output**: Every test must use `lu.assertEquals(msg, expected_output)` with the full expected string, including all color codes, whitespace, and newlines. Never use partial checks like `assertNotNil(msg:find(...))` or `assertTrue(# msg > 0)`.
- **Trailing whitespace handling**: Use `remove_trailing()` helper function when comparing test output to strip trailing spaces from each line. This prevents fragile whitespace comparisons while preserving semantic correctness.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starwing/musubi](https://github.com/starwing/musubi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
