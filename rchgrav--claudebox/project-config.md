---
trigger: always_on
description: - @~/.claudebox/tooling.md
---

## Current Environment Tooling
- @~/.claudebox/tooling.md

## Default Coding Behavior

**CRITICAL: Always consider TDD discipline before starting any coding task**

1. **Before any implementation work** - Ask: "Should I use /tdd mode for this coding task?"
2. **Default to TDD discipline** unless explicitly told otherwise or for trivial changes
3. **TDD applies to most coding scenarios**:
   - Adding new functions or features
   - Implementing validation logic
   - Building utilities or libraries
   - Refactoring with behavioral changes
   - Any multi-step implementation work
4. **Skip TDD only for**:
   - Simple documentation updates
   - Obvious typo fixes
   - Configuration file changes
   - Single-line code adjustments
5. **TDD workflow activation**:
   - Use `/tdd [task description]` command to activate strict discipline
   - Follow red-green-refactor cycle rigorously
   - Never write multiple test cases without implementing each one first
   - Commit test + implementation + docs together for each test case

**Key behavioral change**: Make TDD consideration automatic, not optional.

## Automatic Commit Workflow

### Pre-commit Checks (in order)

**CRITICAL: Always run these checks before any commit:**

1. **Formatting** - Run code formatters first (prettier, black, rustfmt, etc.)
2. **Linting** - Run linters after formatting
3. **Type checking** - Run type checkers
4. **Tests** - Run relevant tests last
5. **Test coverage verification** - Confirm all expected test files are running
6. **All tests must pass** - **CRITICAL**: Fix any failing tests immediately, do not commit/push with failing tests
7. **Final review** - Check `git diff --staged` to review what will be committed
8. **Security check** - Verify no sensitive information (keys, tokens, passwords) is included

### Test Requirements

**All tests must pass before any commit or push:**

1. **Fix failing tests immediately** - Never leave failing tests for "future PRs" or "follow-up work"
2. **CI requirement** - Most CI/CD systems require all tests to pass before merge
3. **Quality gate** - Failing tests indicate broken functionality that must be addressed
4. **No exceptions** - Even if failure seems minor or unrelated, investigate and fix

**When tests fail:**
- **Investigate the root cause** - Don't just change the test, understand why it's failing
- **Fix the implementation or test** - Address the actual issue, whether in code or test logic
- **Verify the fix** - Run the full test suite to ensure no regressions
- **Document complex fixes** - If the fix was non-obvious, add comments explaining the solution

### When Pre-commit Checks Fail

- **Formatting failures**: Auto-fix and stage the formatted changes, then retry commit
- **Linting failures**: Fix the issues, stage the fixes, then retry commit
- **Type checking failures**: Fix type errors, stage the fixes, then retry commit
- **Test failures**: Fix failing tests, stage the fixes, then retry commit
- If any check fails twice, report the issue and ask for guidance
- Always include auto-fixes in the same commit when possible

## Git Workflow

### Commit Strategy

- Create logical, atomic commits that can be reviewed independently
- Each commit should represent a single conceptual change
- Commit related changes together (e.g., function + tests + documentation)
- Separate refactoring commits from feature commits
- Use descriptive commit messages that explain the "why" not just the "what"

### Development Commit Frequency

- **Commit early and often** during feature development
- Make a commit after completing each logical unit of work:
  - Adding a single function with its test
  - Implementing one specific feature or validation
  - Adding documentation for a single component
  - Fixing one specific issue or bug
- **Never bundle unrelated changes** in a single commit
- **One behavior per commit** - each commit should implement exactly one piece of functionality
- **Separate commits even for the same file type** - configuration changes, documentation updates, and code changes should be separate commits even if they modify similar file types
- Prefer 10-20 micro-commits over 3-5 larger commits for a feature
- Each commit should leave the codebase in a working state

### Documentation Updates

Include necessary documentation updates in the same commit as the code change:

- **Update code comments** when changing function behavior or adding parameters
- **Update README.md** if adding new setup steps, dependencies, or usage instructions
- **Update existing examples** that would be invalidated by the change
- **Skip excessive documentation** that would quickly become outdated
- **Focus on user-facing changes** that affect how people use the code

Examples:
- ✅ Adding a new CLI flag? Update README.md usage examples in the same commit
- ✅ Changing function parameters? Update the function's comment block
- ✅ Adding a new dependency? Update installation instructions
- ❌ Don't document internal implementation details that change frequently
- ❌ Don't add verbose explanations for self-documenting code

### TDD Commit Strategy

For detailed TDD workflow enforcement, use the `/tdd` command.

Key principles when following Test-Driven Development:
- One test case at a time with minimal implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RchGrav/claudebox](https://github.com/RchGrav/claudebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
