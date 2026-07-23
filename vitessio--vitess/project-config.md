---
trigger: always_on
description: **We're building this together.** You're not just executing tasks - you're helping design and implement the best possible solution. This means:
---

## :handshake: Our Partnership

**We're building this together.** You're not just executing tasks - you're helping design and implement the best possible solution. This means:

- Challenge my suggestions when something feels wrong
- Ask me to explain my reasoning
- Propose alternative approaches
- Take time to think through problems

**Quality is non-negotiable.** We'd rather spend an hour designing than 3 hours fixing a rushed implementation.

## :thought_balloon: Before We Code

Always discuss first:
- What problem are we solving?
- What's the ideal solution?
- What tests would prove it works?
- Are we making the codebase better?

## Strict Task Adherence

**Only do exactly what I ask for - nothing more, nothing less.**

- Do NOT add explanatory comments unless asked
- Do NOT make "improvements" or "clean up" code beyond the specific task
- Do NOT add features, optimizations, or enhancements I didn't mention
- If there is something you think should be done, suggest it, but don't do it until asked to

**Red flags that indicate you're going beyond the task:**
- "Let me also..."
- "While I'm at it..."
- "I should also update..."
- "Let me improve..."
- "I'll also clean up..."

**If the task is complete, STOP. Don't look for more work to do.**

## :test_tube: Test-Driven Development

TDD isn't optional - it's how we ensure quality:

### The TDD Cycle
1. **Red** - Write a failing test that defines success
2. **Green** - Write minimal code to pass
3. **Refactor** - Make it clean and elegant

### Example TDD Session
```go
// Step 1: Write the test first
func TestConnectionBilateralCleanup(t *testing.T) {
    // Define what success looks like
    client, server := testutils.CreateConnectedTCPPair()
    
    // Test the behavior we want
    client.Close()
    
    // Both sides should be closed
    assert.Eventually(t, func() bool {
        return isConnectionClosed(server)
    })
}

// Step 2: See it fail (confirms we're testing the right thing)
// Step 3: Implement the feature
// Step 4: See it pass
// Step 5: Refactor for clarity
```

To make sure tests are easy to read, we use `github.com/stretchr/testify/assert` and `github.com/stretchr/testify/require` for assertions:
- Use `require` (not `assert`) when the test cannot continue after a failure (e.g., `require.NoError` after setup that must succeed)
- Use `assert.Eventually` instead of manual `time.Sleep()` and timeouts
- Use `t.Context()` instead of `context.Background()` — it integrates with test cancellation
- Use `t.Cleanup()` for test teardown
- Use `assert.ErrorContains` / `require.ErrorContains` to check error messages
- Use the `_test.go` suffix for mocks and test helpers that are only used by the current package's tests; if helpers or mocks need to be imported by other packages' tests or fuzz harnesses, put them in a normal reusable package such as `testlib` or `testutil`
- CI timeouts must be generous (30s+) — GitHub Actions runners can be resource-starved with multi-second pauses; sub-second timeouts cause flakiness with no recourse but retry
- Do not use t.Fatal or t.Error in tests, but instead require and assert

### Test Honesty
- A test must actually exercise the condition its name and doc claim, must fail on `main` without the fix it guards, and must not duplicate coverage that a unit test already pins down precisely. Tests that pass identically with or without the fix waste CI time and create false confidence.

## :rotating_light: Error Handling Excellence

Error handling is not an afterthought - it's core to reliable software.

### Go Error Patterns
```go
// YES - Clear error context with vterrors
func ProcessUser(id string) (*User, error) {
    if id == "" {
        return nil, vterrors.Errorf(vtrpcpb.Code_FAILED_PRECONDITION, "user ID cannot be empty")
    }

    user, err := db.GetUser(id)
    if err != nil {
        return nil, vterrors.Wrapf(err, "failed to get user %s", id)
    }

    return user, nil
}

// NO - Swallowing errors
func ProcessUser(id string) *User {
    user, _ := db.GetUser(id)  // What if this fails?
    return user
}
```

### Error Handling Principles
1. **Use `vterrors`** - Prefer `vterrors` over `fmt.Errorf` or `errors` package, with an appropriate `vtrpcpb.Code` (e.g., `vtrpcpb.Code_FAILED_PRECONDITION` for unexpected input values, `vtrpcpb.Code_INTERNAL` for internal operation failures)
2. **Wrap errors with context** - Use `vterrors.Wrapf(err, "context")`
3. **Validate early** - Check inputs before doing work
4. **Fail fast** - Don't continue with invalid state
5. **Log appropriately** - Errors at boundaries, debug info internally
6. **Return structured errors** - Use error types for different handling
7. **Never silently swallow errors** - When recovering from an error (e.g., restarting replication), always log the original error before the recovery attempt so operators can trace what happened
8. **Log with context** - Include workflow name, recovery type, tablet alias, and other identifiers in log messages — a keyspace/tablet can have many concurrent workflows

### Failure-Path Safety
Multi-step operations must not leave the system in a half-applied state:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vitessio/vitess](https://github.com/vitessio/vitess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
