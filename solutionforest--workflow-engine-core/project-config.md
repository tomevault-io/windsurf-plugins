---
trigger: always_on
description: Instructions for AI agents working on this codebase. Read CLAUDE.md first for project context.
---

# AGENTS.md

Instructions for AI agents working on this codebase. Read CLAUDE.md first for project context.

---

## General Rules

1. **Read before you write.** Never modify a file you haven't read. Understand the existing patterns.
2. **Zero production dependencies.** This is a framework-agnostic library. Never add runtime `require` entries to composer.json. Dev dependencies are fine.
3. **PHP 8.3+ only.** Use readonly properties, backed enums, match expressions, named arguments, constructor promotion. No legacy patterns.
4. **Run the full quality pipeline** after any change: `composer ci` (pint:test + analyze + test).
5. **PHPStan level 6.** All new code must pass. Don't suppress errors unless truly unavoidable—and document why.
6. **Pest PHP for tests.** Use `test()` syntax, `expect()` assertions, `describe()` blocks. No raw PHPUnit `$this->assert*()`.
7. **Laravel Pint formatting.** Run `composer pint` before committing. The CI will reject improperly formatted code.

---

## Agent: Code Review

When reviewing PRs or code changes for this project:

### What to Check

- **Contract compliance**: Does new code implement `WorkflowAction`, `StorageAdapter`, `EventDispatcher`, or `Logger` correctly?
- **Immutability**: `WorkflowContext`, `ActionResult`, `WorkflowDefinition` are value objects. Don't add setters or mutation methods.
- **State transitions**: Any code that calls `setState()` must respect the transition rules in `WorkflowState::canTransitionTo()`. Invalid transitions are bugs.
- **Action instantiation**: Actions are created with `new $actionClass($config, $logger)`. If you change the `WorkflowAction` interface or `BaseAction` constructor, you must update `Executor::executeAction()`.
- **No framework coupling**: This package must not `use` any Laravel, Symfony, or other framework classes in `src/`. The only exception is `data_get()`/`data_set()` helpers (which need to be replaced—see Known Issues).
- **Exception hierarchy**: All exceptions must extend `WorkflowException`. Use the static factory methods (e.g., `InvalidWorkflowDefinitionException::invalidName()`), not raw `new` constructors.
- **Event consistency**: Every state-changing operation should dispatch the appropriate event. Check that `WorkflowStarted`, `WorkflowCompleted`, `WorkflowFailed`, `WorkflowCancelled`, `StepCompleted`, `StepFailed` are dispatched at the right moments.

### What to Reject

- Adding `declare(strict_types=1)` piecemeal—if we add it, it goes in every file at once.
- Magic string config keys without constants or typed config objects.
- Suppressing PHPStan errors without a clear comment explaining why.
- Tests that use `assertTrue(true)` or other vacuous assertions.
- New public API methods without PHPDoc `@param`, `@return`, and `@throws` tags.

---

## Agent: Testing

### Test Structure

```
tests/
├── Unit/              # Isolated class tests (no I/O, no storage)
├── Integration/       # Multi-step workflow execution through the engine
├── RealWorld/         # Complex production-like scenarios
├── Actions/ECommerce/ # Custom action fixtures used by RealWorld tests
├── Support/           # Test helpers (InMemoryStorage)
├── TestCase.php       # Base class: provides $this->engine + $this->storage
├── Pest.php           # Pest config
├── ArchTest.php       # Architecture constraints
└── ExampleTest.php    # Sanity check
```

### Writing Tests

```php
// Good: descriptive, focused, uses Pest syntax
test('workflow transitions to failed state when action throws', function () {
    $definition = [
        'name' => 'failing-workflow',
        'steps' => [
            ['id' => 'bad_step', 'action' => NonExistentAction::class],
        ],
    ];

    expect(fn () => $this->engine->start('test-fail', $definition))
        ->toThrow(ActionNotFoundException::class);
});

// Good: grouped with describe
describe('WorkflowBuilder', function () {
    test('validates step IDs', function () { ... });
    test('rejects empty workflow names', function () { ... });
});
```

### Test Gaps to Fill

These areas currently lack test coverage. Prioritize them when writing new tests:

1. **Event dispatch verification** — No tests confirm events are actually dispatched. Create a `SpyEventDispatcher` that records dispatched events and assert against it.
2. **Retry logic** — `Step` supports `retryAttempts` but the `Executor` never actually retries. When retry is implemented, add tests for 0, 1, and max retries.
3. **HTTP/Email actions** — `HttpAction` and `EmailAction` have no unit tests. Mock the underlying operations and test config validation, error handling, and result mapping.
4. **Storage adapter edge cases** — Only `InMemoryStorage` is tested. Add tests for: loading a non-existent instance, concurrent saves, findInstances with various filter combinations.
5. **Condition evaluation** — The regex-based condition parser in `Step::evaluateCondition()` silently returns `true` for unparseable conditions. Test edge cases: nested properties, numeric comparisons, boolean values, empty strings.
6. **Compensation actions** — `Step` supports `compensationAction` but nothing executes it. When implemented, test rollback sequences.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solutionforest/workflow-engine-core](https://github.com/solutionforest/workflow-engine-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
