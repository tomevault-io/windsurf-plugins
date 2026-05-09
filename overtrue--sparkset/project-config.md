---
trigger: always_on
description: - **Absolutely prohibit executing git reset, git revert, git rebase, git restore, and other rollback commands**
---

# Development Guidelines

## Strict Prohibited Operations

### Git Operation Restrictions

- **Absolutely prohibit executing git reset, git revert, git rebase, git restore, and other rollback commands**
- **Only allow safe operations like git logs, git status, git diff to compare files and restore content**
- **Prohibit deleting or modifying the .git directory**
- **Any git operation must get explicit user permission first**

### File System Operation Restrictions

- **Absolutely prohibit executing rm -rf command**
- **Prohibit deleting directories, especially project root or important directories**
- **Must clearly inform user and get permission before deleting files**

## Communication Language

**Important**: Please use the same language with the user in your communication and responses. This includes:

- All conversations and responses
- Code comments (unless project specification requires English)
- Documentation and explanations
- Error messages and explanations
- Task plans and summaries

## Philosophy

### Core Beliefs

- **Incremental progress over big bangs** - Small changes that compile and pass tests
- **Learning from existing code** - Study and plan before implementing
- **Pragmatic over dogmatic** - Adapt to project reality
- **Clear intent over clever code** - Be boring and obvious
- **No backward compatibility required** - This is a new project, all changes can be breaking changes

### Simplicity Means

- Single responsibility per function/class
- Avoid premature abstractions
- No clever tricks - choose the boring solution
- If you need to explain it, it's too complex

## Process

### 1. Planning & Staging

Break complex work into 3-5 stages. Document in `IMPLEMENTATION_PLAN.md`:

```markdown
## Stage N: [Name]

**Goal**: [Specific deliverable]
**Success Criteria**: [Testable outcomes]
**Tests**: [Specific test cases]
**Status**: [Not Started|In Progress|Complete]
```

- Update status as you progress
- Remove file when all stages are done

### 2. Implementation Flow

1. **Understand** - Study existing patterns in codebase
2. **Test** - Write test first (red)
3. **Implement** - Minimal code to pass (green)
4. **Refactor** - Clean up with tests passing
5. **Validate** - Ensure compilation and tests run
6. **Update TODO** - Mark completed tasks and summarize achievements
7. **Commit** - With clear message linking to plan

**Key**: After code compiles successfully, always:

- Update TODO list to mark completed tasks
- Add summary of completed content
- Plan next steps (if applicable)
- Never let TODO list become outdated or stagnant

### 3. When Stuck (After 3 Attempts)

**Critical**: Maximum 3 attempts per issue, then STOP.

1. **Document what failed**:
   - What you tried
   - Specific error messages
   - Why you think it failed

2. **Research alternatives**:
   - Find 2-3 similar implementations
   - Note different approaches used

3. **Question fundamentals**:
   - Is this the right abstraction level?
   - Can this be split into smaller problems?
   - Is there a simpler approach entirely?

4. **Try different angle**:
   - Different library/framework feature?
   - Different architectural pattern?
   - Remove abstraction instead of adding?

## Technical Standards

### Architecture Principles

- **Composition over inheritance** - Use dependency injection
- **Interfaces over singletons** - Enable testing and flexibility
- **Explicit over implicit** - Clear data flow and dependencies
- **Test-driven when possible** - Never disable tests, fix them
- **No backward compatibility** - This is a new, unreleased project. All refactoring can be breaking changes. Don't maintain deprecated code or compatibility layers unless absolutely necessary

### Code Quality

- **Every commit must**:
  - Compile successfully
  - Pass all existing tests
  - Include tests for new functionality
  - Follow project formatting/linting

- **Before committing**:
  - Run formatters/linters
  - Self-review changes
  - Ensure commit message explains "why"

### Error Handling

- Fail fast with descriptive messages
- Include context for debugging
- Handle errors at appropriate level
- Never silently swallow exceptions

### Refactoring Principles

**No Backward Compatibility Required**: This is a new, unreleased project. When refactoring:

- **Remove deprecated code directly** - Don't maintain compatibility layers or deprecated functions
- **Break existing patterns if needed** - If a better pattern exists, migrate all code to it
- **Don't keep old implementations** - Remove old code instead of keeping it "just in case"
- **Update all usages immediately** - When changing an API or pattern, update all call sites
- **No deprecation warnings needed** - Since this is a new project, just remove and replace

**Exception**: Only maintain compatibility if explicitly required by external dependencies or critical business needs.

### Compilation Error Handling

**Fundamental Principle**: Never delete code to bypass compilation errors. Fix the root cause.

When encountering compilation errors:

1. **NEVER do this**:
   - Delete problematic methods/code
   - Comment out error lines
   - Use placeholder implementations (TODO, throw NotImplemented)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [overtrue/sparkset](https://github.com/overtrue/sparkset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
