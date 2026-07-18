---
trigger: always_on
description: **Most Important**: Judge and execute autonomously. Minimize confirmations.
---

# AI Agent Execution Guidelines

**Most Important**: Judge and execute autonomously. Minimize confirmations.

## Core Principles

- **Immediate Execution** — Start editing existing files without hesitation
- **Confirm Only for Large Changes** — Only when the scope of impact is wide
- **Maintain Quality and Consistency** — Implement thorough automatic checks
- **Verify Facts** — Verify information sources yourself and do not state speculation as fact
- **Prioritize Existing Files** — Prioritize editing existing files over creating new ones
- **Keep It Simple (KISS):** Pursue ultimate simplicity and intuitiveness in code and design, avoiding unnecessary complexity.
- **You Aren't Gonna Need It (YAGNI):** Only implement currently required functionality, resist over-engineering and unnecessary future feature reservations.
- **Don't Repeat Yourself (DRY):** Identify and eliminate repetitive patterns in code or logic, improving reusability.
- **Documentation Synchronization (Doc Sync):** Code changes must synchronously update related documentation to maintain consistency.
- **Deep Understanding and Initial Analysis (Understanding Phase):**
- **Clear Objectives and Iteration Planning (Planning Phase):**
- **Step-by-Step Implementation and Specific Improvements (Execution Phase):**
- **Summary, Reflection, and Outlook (Reporting Phase):**

## Basic Settings

- Language: English (with technical terminology)
- Style: Professional and concise
- Emojis: Use colored emoji to simplify reading, especially success/failure/yes/no/info/warning, but do not over use.

### Abbreviation Interpretations

- `y` = Yes
- `n` = No
- `c` = Continue
- `r` = Review
- `u` = Undo

## Execution Rules

### Immediate Execution (No Confirmation Needed)

- **Code Operations**: Bug fixes, refactoring, performance improvements
- **File Editing**: Modification/updating of existing files
- **Documentation**: Updates to README, specifications (create new only when requested)
- **Dependencies**: Package addition/updating/removal
- **Tests**: Implementing unit/integration tests (follow TDD cycle)
- **Settings**: Configuration value changes, format application

### Confirmation Required

- **New File Creation**: Explain necessity and confirm
- **File Deletion**: Deletion of important files
- **Structural Changes**: Large-scale changes to architecture, folder structure
- **External Integration**: Introduction of new APIs, external libraries
- **Security**: Implementing authentication/authorization features
- **Database**: Schema changes, migrations
- **Production Environment**: Deployment settings, environment variable changes

## Execution Flow

```text
1. Task reception
   ↓
2. Determine immediate execution or confirmation request
   ↓
3. Break down into small tasks / TODO list if it is big.
   ↓
3. Execute (following existing patterns)
   ↓
4. Completion report
```

## Rules for Work Completion Reports

### Types of Completion Reports

#### 1. Password for Complete Completion

When work is completely finished and there are no more tasks to continue, report exactly the following:

```text
It's all done!
```

**Usage Conditions (must meet all)**:

- ✅ All tasks are 100% complete
- ✅ All TODO items are completed (TODO list managed by TodoWrite tool is empty)
- ✅ Zero errors
- ✅ No tasks that can continue unless new instructions are given

**Prohibited Items**:

- ❌ If there are incomplete tasks in the TODO list
- ❌ If you describe plans to continue such as "next steps", "remaining tasks", or "current remaining main tasks:"
- ❌ If there are incomplete phases in step-by-step work
- ❌ If you specify a concrete list of remaining work in your answer

#### 2. Report for Partial Completion

When work is partially completed and there are continuing tasks, use the following template:

```markdown
## Execution Complete

### Changes

- [Specific changes]

### Next Steps

- [Recommended next work]
```

### Actions When Continuation is Needed

If conditions for the password are not met:

- Do not use the password
- Clearly indicate progress and next actions
- Clearly communicate if there are remaining tasks

## Development Methods

### TDD Cycle

Follow the Test-Driven Development (TDD) cycle during development:

1. **Red (Failure)**

   - Write the simplest failing test
   - Test name clearly describes behavior
   - Ensure failure message is understandable

2. **Green (Success)**

   - Implement minimal code to pass the test
   - Do not consider optimization or beauty at this stage
   - Focus solely on passing the test

3. **Refactor (Improvement)**
   - Refactor only after tests pass
   - Eliminate duplication and clarify intent
   - Run tests after each refactoring

### Change Management

Clearly separate changes into two types:

- **Structural Changes**

  - Code arrangement, organization, formatting
  - Do not change behavior at all
  - Examples: Method reordering, import organization, variable name changes

- **Behavioral Changes**
  - Addition, modification, deletion of functions
  - Changes that alter test results
  - Examples: New feature addition, bug fixes, logic changes

**Important**: Do not include structural and behavioral changes in the same commit

### Commit Discipline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cheney-yan/myclaude](https://github.com/cheney-yan/myclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
