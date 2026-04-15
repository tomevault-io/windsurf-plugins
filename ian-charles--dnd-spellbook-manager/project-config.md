---
trigger: always_on
description: Every line of code and every dependency is a liability. Write for clarity, maintainability, and performance. Fast is a feature.
---

# Gemini Development Guidelines

## Core Philosophy

Every line of code and every dependency is a liability. Write for clarity, maintainability, and performance. Fast is a feature.

## Technical Design Documents

Create a design doc for any work > 1 week or that affects architecture, performance, or APIs.

**Template**: Problem Statement → Goals/Non-Goals → Solution (architecture, API, data model) → Performance considerations → Alternatives → Testing/Rollout → Timeline

## TDD Workflow

### Red-Green-Refactor
1. **Red**: Write failing test → `git commit -m "test: add test for [feature]"`
2. **Green**: Minimal code to pass → `git commit -m "feat: implement [feature]"`  
3. **Refactor**: Improve quality → `git commit -m "refactor: [improvement]"`

**Principles**: One assertion per test • Test behavior, not implementation • AAA pattern • Fast tests first (unit > integration > e2e)

## Git Commits

### Format
`<type>(<scope>): <subject>`

### Types
- `feat`: New feature
- `fix`: Bug fix
- `test`: Add or modify tests
- `refactor`: Code refactoring (no behavior change)
- `docs`: Documentation only
- `perf`: Performance improvement
- `chore`: Build process, dependencies, etc.

### Rules
1. **Always run tests before committing** - `npm test` must pass
2. **NEVER bypass or kill git hooks** - Hooks exist for quality control:
   - NEVER use `--no-verify` to bypass hooks
   - NEVER kill hook processes (pkill, Ctrl+C, etc.)
   - NEVER disable or remove hooks to commit faster
   - NEVER modify git hooks unless explicitly instructed to do so
   - If pre-commit hook fails, fix the issue, don't bypass it
   - If tests are slow, optimize them, don't skip them
   - If hooks are broken, fix the hooks, don't bypass them
   - Bypassing hooks = shipping broken code to production
   - Killing hooks = worse than bypassing (no record of what was skipped)
3. **Commit after each meaningful change**:
   - After test passes (RED → GREEN)
   - After refactoring (GREEN → REFACTOR)
   - After each complete feature
4. **Single purpose per commit**:
   - Each commit should have ONE clear purpose
   - If a commit has multiple unrelated changes, split it into multiple commits
   - Examples: Don't mix "fix bug" with "add feature" in same commit
5. **Never commit**:
   - Broken tests
   - Commented-out code
   - Debugging console.log statements
   - Files with secrets (.env, credentials.json)
6. **Write clear messages**:
   - Subject: What changed (imperative mood: "add", not "added")
   - Body: Why it changed (optional but recommended)

### Examples
```bash
# Good commits
git commit -m "test: add test for mobile spell expansion"
git commit -m "fix: React Fragment key warning preventing mobile expansion"
git commit -m "refactor: extract scroll behavior to helper function"

# Bad commits
git commit -m "fix stuff"  # Too vague
git commit -m "WIP"  # Work in progress shouldn't be committed
git commit -m "updated code"  # Doesn't explain what changed
```

### Before Every Commit
```bash
# Run tests
npm test

# Check what you're committing
git diff --cached

# Verify no debug code
git diff --cached | grep -i "console.log\|debugger\|TODO"
```

## Code Style

### Self-Documenting Code
- **Names reveal intent**: `getUserByEmail()` not `getUser()` or `gube()`
- **Functions do one obvious thing**: No surprises, no side effects
- **Variables explain themselves**: `daysUntilExpiration` not `d` or `days`
- **Code structure tells the story**: Logical flow that reads top-to-bottom

### Naming: Scope Matters

Names inherit context from their enclosing scopes. The tighter the scope, the shorter the name can be.

**Naming by scope:**
- **Single letter** (`i`, `j`, `n`): Loop indices, very short functions
- **Short** (`user`, `err`, `ctx`): Function parameters, local variables
- **Descriptive** (`activeUsers`, `retryCount`): Function/method level
- **Fully qualified** (`MaxRetryAttempts`, `DefaultTimeoutSeconds`): Package/global level

```go
// Bad: Overly verbose in tight scope
for customerIndex := 0; customerIndex < len(customers); customerIndex++ {
    processCustomer(customers[customerIndex])
}

// Good: Short name in tight scope
for i, c := range customers {
    processCustomer(c)
}
```

```python
# Good: Progressive naming based on scope
def calculate_total(orders):
    t = 0  # 't' is fine - function name provides context
    for o in orders:
        t += o.amount
    return t
```

**Function constraints**: < 20 lines, < 3 parameters, single responsibility  
**Structure**: Early returns, UPPER_SNAKE_CASE constants, DRY, YAGNI

## Comments: Self-Documenting Code First

**Primary goal**: Write code so clear that comments become unnecessary. If you need to explain WHAT the code does, rewrite the code.

### When Comments ARE Needed
- **Public APIs**: Document contracts and usage
- **Non-obvious business rules**: Include ticket/requirement references  
- **Critical warnings**: Consequences that aren't apparent
- **Performance trade-offs**: Why you chose this approach
- **Workarounds**: Temporary fixes with removal timeline

### ❌ Never Write
```python
i += 1  # increment i (obvious)
# old_function() (commented-out code)
#--------- GETTERS --------- (noise)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ian-charles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
