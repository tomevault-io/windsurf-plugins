---
trigger: always_on
description: **ALWAYS check this rules file FIRST before doing anything.** This file contains critical information about:
---

# Cursor Rules for pywiim

## FIRST: Read This File

**ALWAYS check this rules file FIRST before doing anything.** This file contains critical information about:
- Project structure (where files go)
- Environment setup (how to run code)
- Testing procedures (how to test)
- Design patterns (how to implement)

## MANDATORY: Before Implementing ANY Fix or Feature

**STOP. Do NOT propose solutions until you complete this checklist:**

1. ☐ **Read design docs FIRST** - Before suggesting ANY implementation approach:
   - `docs/design/API_DESIGN_PATTERNS.md` - How we handle device variations
   - `docs/design/DESIGN_PRINCIPLES.md` - Our architectural patterns
   - `docs/design/ARCHITECTURE.md` - System structure and data flow
   - `docs/design/adr/` - Architecture Decision Records (decisions we have committed to; check before changing behavior that might be covered)

2. ☐ **Find similar patterns** - Search the codebase:
   - How does existing code handle similar device variations?
   - Is there a capability/profile pattern already handling this type of problem?
   - What existing pattern should this follow?

3. ☐ **Propose approach** - Discuss with user BEFORE coding:
   - Explain which existing pattern applies
   - Show how the fix follows established architecture
   - Get user approval before implementation

4. ☐ **Implement** - Only after steps 1-3 are complete

**WHY THIS MATTERS:** Proposing solutions without reading design docs leads to:
- Suggesting patterns that violate our architecture
- Creating inconsistencies that need to be refactored later
- Wasting time discussing approaches that don't fit

## Core Principles

1. **Check the rules file first** - Read this file before implementing, testing, or creating anything.

2. **Follow the design guide** - Always check `docs/design/` for design patterns and API standards before implementing anything.

3. **Follow the API standard** - Implement according to the documented API patterns, not made-up solutions.

4. **Do as asked** - Implement exactly what the user requests, nothing more, nothing less.

5. **Ask if uncertain** - If you don't know what to do or are uncertain about the approach, ASK THE USER before implementing anything.

6. **No made-up solutions** - Do not invent complex logic or "solutions" that aren't explicitly requested. Stick to the design guide and API standards.

7. **Stop and ask when things get complicated** - If something isn't working, doesn't make sense, or seems to require increasingly complex workarounds:
   - **STOP** - Do not continue hacking together fixes
   - **Do NOT create new files** as workarounds or "extra documentation"
   - **Do NOT pile on hack after hack** to make broken code work
   - **Instead**: Step back, review the design patterns, identify what's actually wrong, and ASK THE USER how to proceed
   - If you find yourself writing convoluted logic or creating workaround files, that's a signal to stop and ask
   - A simple question is always better than a complex hack
   - **First check**: "How does similar working code handle this?" (See "Before Adding Special Cases" section)

## When Implementing

- Check the design guide first (`docs/design/API_DESIGN_PATTERNS.md` and related files)
- Follow existing patterns in the codebase
- If the design guide says "do X", do X - don't add extra complexity
- If you're not sure, ask the user
- **Significant decisions** (user-facing contracts, naming stability, compatibility promises): record in an ADR in `docs/design/adr/`; see `docs/design/adr/README.md` for format and when to write one

## Testing Requirements

**CRITICAL: New code MUST have tests.**

1. **All new functionality requires tests**
   - New methods/functions → Add unit tests in `tests/unit/`
   - Bug fixes → Add regression test that would have caught the bug
   - API format changes → Add integration test that verifies actual API behavior
   - New normalization logic → Add parameterized tests covering all variations

2. **Unit tests verify logic, not just that code runs**
   - Mock external dependencies, but verify the VALUES passed to mocks
   - Test edge cases and error paths
   - Example: If testing `set_source()`, verify the EXACT string sent to the API

3. **Integration tests verify real-world behavior**
   - Tests in `tests/integration/` run against real devices
   - Use `WIIM_TEST_DEVICE=<ip>` to enable
   - Critical for API format verification (mocks can't catch wrong formats)

4. **Run tests before considering work complete**
   - `make test` - Run all unit tests
   - `WIIM_TEST_DEVICE=<ip> pytest tests/integration/` - Run integration tests
   - Fix any test failures before committing

5. **Test coverage for bug fixes**
   - Ask: "What test would have caught this bug?"
   - Write that test FIRST, verify it fails, then fix the bug
   - This prevents the same bug from recurring

## Before Adding Special Cases or Fixes

**CRITICAL: Always check existing patterns first before adding special cases or fixes.**

1. **Question the problem statement** - Don't assume the problem description is correct
   - If an issue says "X doesn't work", verify: "Is X using the wrong method/endpoint?"
   - Check: "Are we using the same pattern that works elsewhere?"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mjcumming/pywiim](https://github.com/mjcumming/pywiim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
