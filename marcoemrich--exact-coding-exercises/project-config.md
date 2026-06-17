---
trigger: always_on
description: Core TDD workflow rules - Red-Green-Refactor cycle with specialized agents
---


# Test-Driven Development (TDD) Rules

## CRITICAL: Agent Usage is MANDATORY

**YOU MUST USE THE SPECIALIZED TDD AGENTS FOR EVERY TDD TASK.**

Do NOT perform TDD phases manually. The agents enforce discipline and prevent common mistakes.

### Before Starting Any TDD Work - Complete This Checklist:

- [ ] Have I been asked to implement something using TDD?
- [ ] Am I about to write tests or implementation code?
- [ ] **STOP** - Use the Task tool to launch the appropriate TDD agent
- [ ] NEVER write tests or code directly - ALWAYS use agents

### Which Agent to Use:

| Phase | Agent Name | Use Task Tool With |
|-------|-----------|-------------------|
| Test List | `test-list` | `subagent_type: "test-list"` |
| Red Phase | `red` | `subagent_type: "red"` |
| Green Phase | `green` | `subagent_type: "green"` |
| Refactor Phase | `refactor` | `subagent_type: "refactor"` |

**If you find yourself writing test code or implementation code without launching an agent first, you are doing it WRONG.**

## TDD Workflow

**MANDATORY: Use the specialized TDD agents for each phase of the cycle:**

### 1. Test List Phase
**LAUNCH AGENT**: Use `Task` tool with `subagent_type: "test-list"`

**Required prompt context:**
- Feature/function to implement
- Target file paths (test file + implementation file)
- Any constraints or requirements from the user

**Example Task call:**
```
Task({
  subagent_type: "test-list",
  prompt: `
    Feature: String Calculator
    Test file: src/calculator.spec.ts
    Implementation file: src/calculator.ts
    Requirements: Parse comma-separated numbers and return sum
  `
})
```

The agent will create a comprehensive test list using `it.todo()` for BASE FUNCTIONALITY ONLY:
- Focus on core behavior, not advanced features
- Order tests from simple to complex
- No implementation yet

**DO NOT** write the test list yourself - let the agent do it.

### 2. Red Phase
**LAUNCH AGENT**: Use `Task` tool with `subagent_type: "red"`

**Required prompt context:**
- Test file path
- Which `it.todo()` to activate (name or line number)
- Current state (number of passing tests)
- Implementation file path

**Example Task call:**
```
Task({
  subagent_type: "red",
  prompt: `
    Test file: src/calculator.spec.ts
    Activate test: "should return sum for two numbers" (line 12)
    Current state: 2 tests passing
    Implementation file: src/calculator.ts
  `
})
```

The agent will activate exactly ONE test and make it fail:
- Convert one `it.todo()` to executable test
- Make explicit predictions (Guessing Game)
- Verify compilation error, then runtime error
- **Stop and wait for approval** before Green phase

**DO NOT** write test code yourself - let the agent do it.

### 3. Green Phase
**LAUNCH AGENT**: Use `Task` tool with `subagent_type: "green"`

**Required prompt context:**
- Test file path
- Failing test name and expected behavior
- Current error message
- Implementation file path

**Example Task call:**
```
Task({
  subagent_type: "green",
  prompt: `
    Test file: src/calculator.spec.ts
    Failing test: "should return sum for two numbers"
    Expected: add("1,2") returns 3
    Current error: Expected 3, Received undefined
    Implementation file: src/calculator.ts
  `
})
```

The agent will implement minimal code to make the test pass:
- Use the simplest possible solution
- Hardcoded returns are acceptable early on
- No features for future tests
- **Stop and wait for approval** before Refactor phase

**DO NOT** write implementation code yourself - let the agent do it.

### 4. Refactor Phase
**LAUNCH AGENT**: Use `Task` tool with `subagent_type: "refactor"`

**CRITICAL: Refactor MUST be a separate agent call.** The orchestrating agent must NEVER combine Green and Refactor into one Task. After Green phase completes, launch a NEW Task with `subagent_type: "refactor"` — do NOT ask the Green agent to refactor. Each phase = one dedicated agent invocation.

**Required prompt context:**
- Test file path
- Implementation file path
- Current number of passing tests
- Recent changes made in Green phase

**Example Task call:**
```
Task({
  subagent_type: "refactor",
  prompt: `
    Test file: src/calculator.spec.ts
    Implementation file: src/calculator.ts
    Passing tests: 3
    Recent Green phase: Added split/map/reduce for comma parsing

    Refactor the implementation while keeping all tests green.
  `
})
```

The agent will improve code while keeping tests green:
- **MUST attempt at least one refactoring**
- Evaluate naming FIRST
- Apply Four Rules of Simple Design (priority order)
- Calculate APP (Absolute Priority Premise) mass
- Document improvements or why none were possible
- **Stop and wait for approval** before next test

**DO NOT** refactor code yourself - let the agent do it.

### 5. Repeat
Return to step 2 (Red phase) for the next test in the list.

**Launch the `red` agent again - DO NOT proceed manually.**

## Core TDD Principles

### TDD Mindset
TDD practices will feel counterintuitive:
- **Hardcoded returns feel "too simple"** - This is correct!
- **The urge to implement ahead is strong** - Resist this
- **Minimal steps feel inefficient** - They actually accelerate development
- **Predictions feel unnecessary** - They build crucial understanding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcoemrich/EXACT-Coding-Exercises](https://github.com/marcoemrich/EXACT-Coding-Exercises) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
