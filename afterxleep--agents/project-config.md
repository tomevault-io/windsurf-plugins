---
trigger: always_on
description: > Rules for AI agents. Be literal. Follow numbered steps. When ambiguous, ask.
---

# AGENTS.md — Engineering Standards

> Rules for AI agents. Be literal. Follow numbered steps. When ambiguous, ask.

---

## Core Engineering Principles

1. **Clarity over cleverness** — Write code that's maintainable, not impressive
2. **Explicit over implicit** — No magic. Make behavior obvious
3. **Composition over inheritance** — Small units that combine
4. **Fail fast, fail loud** — Surface errors at the source
5. **Delete code** — Less code = fewer bugs. Question every addition
6. **Verify, don't assume** — Run it. Test it. Prove it works
7. **Never modify** files you did not create unless the task explicitly requires it.

---

## Feature Development Workflow

### Before Writing Code

1. **Clarify requirements** — Restate the goal. Ask questions if ambiguous.
2. **Identify failure modes** — Identify what can go wrong:
   - Invalid inputs
   - Missing dependencies
   - Network/IO failures
   - Concurrency issues
   - Memory Leaks / Resource exhaustion
3. **Classify work by priority**:
   - **A. Core flow**: Happy path + direct error cases
   - **B. Edge cases**: Unusual but valid scenarios
   - **C. Out of scope**: Document, don't implement
4. **Check existing code** — Is this already solved? Can you extend rather than create?

### Implementation Order

1. Write failing test for core happy path
2. Implement minimum code to pass
3. Write failing tests for core error cases
4. Implement error handling
5. Refactor if needed (tests stay green)
6. Add edge case tests only after core is solid

### Before Submitting

- [ ] All tests pass
- [ ] No commented-out code
- [ ] No TODO without context (`// TODO: [reason] description`)
- [ ] Error messages are actionable
- [ ] No secrets, credentials, or hardcoded environment-specific values
- [ ] Formatting/linting passes

---

## Debugging Process

### When Something Fails

1. **Reproduce reliably** — Can you trigger it consistently?
2. **Isolate the scope** — What's the smallest input that fails?
3. **Read the error** — Actually read it. Full stack trace.
4. **Form a hypothesis** — One specific guess about the cause
5. **Test the hypothesis** — Add logging, write a test, or inspect state
6. **Fix and verify** — Change one thing. Confirm it's fixed.
7. **Add regression test** — Ensure it can't silently break again

### Don't

- Change multiple things at once
- Assume you know the cause without evidence
- Delete error handling to "simplify"
- Fix symptoms instead of root causes

---

## Code Organization

### File Structure

- One type per file
- Filename matches type name exactly
- Group by feature/domain, not by layer
- Shared utilities in `Core/` or `Common/` — zero domain dependencies

### Dependency Direction

```
Features → Services → Core
    ↓          ↓        ↓
   UI      Business   Utilities
           Logic
```

- `Core/` depends on nothing internal
- `Services/` depends only on `Core/`
- `Features/` depends on `Services/` and `Core/`
- No circular dependencies
- Features are deletable without breaking unrelated code

---

## Code Style

### Naming

| Type | Convention |
|------|------------|
| Types/Classes | `UpperCamelCase` |
| Functions/Variables | `lowerCamelCase` |
| Constants | `lowerCamelCase` or `SCREAMING_SNAKE_CASE` (follow language convention) |
| Protocols/Interfaces | `UpperCamelCase`, noun or adjective |

### Functions

- Do one thing
- Name describes what, not how
- Max 3-4 parameters — beyond that, use a config object
- Avoid boolean parameters — they obscure intent at call sites

```
// Bad
build(scheme, true, false)

// Good
build(scheme, configuration: .release, clean: true)
```

### Comments

- Explain WHY, not WHAT
- Delete comments that restate code
- TODO format: `// TODO: [context] description`
- Document non-obvious behavior and workarounds

---

## Error Handling

### Rules

1. Define domain-specific error types per module
2. Include context: what failed, with what inputs
3. Map external errors at boundaries — don't leak implementation details
4. Fail at the source — don't pass invalid state hoping someone handles it
5. Errors are API — design them like success paths

### Error Checklist

- [ ] Message helps diagnose the problem
- [ ] Includes relevant context (IDs, paths, values)
- [ ] Caller can distinguish error types programmatically
- [ ] Transient vs permanent failures are distinguishable

---

## Testing

### Principles

- Tests are isolated: no shared state, no execution order dependencies
- One behavior per test, descriptive names: `test_build_failsWhenSchemeNotFound`
- Tests run in parallel — design for it
- Test behavior, not implementation
- Fast tests get run; slow tests get skipped

### Before Writing a Test

1. Check if the **behavior** is already tested (not just the code path — multiple tests can hit the same lines testing different behaviors)
2. Identify the mock in `Tests/Mocks/` — create only if missing
3. Identify the fixture in `Tests/Fixtures/` — create only if missing

### When Adding Features

1. **Document failure modes first** — List edge cases and failure scenarios in a comment block before writing implementation code:
   ```
   // Failure modes:
   // - Scheme not found in project
   // - Build timeout exceeded  
   // - Simulator unavailable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afterxleep/agents](https://github.com/afterxleep/agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
