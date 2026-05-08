---
trigger: always_on
description: Patterns for writing tests — integration tests with TestDB, unit tests for pure logic, test data seeding
---


# Writing Tests

## Integration Tests (Service Layer)

Test real SQL against a real PostgreSQL instance using `testutil.WithTestDB`.

```go
//go:build integration

package service_test

import (
    "testing"
    "github.com/golid-ai/golid/backend/internal/service"
    "github.com/golid-ai/golid/backend/internal/testutil"
)

func TestItemService_Create(t *testing.T) {
    testutil.WithTestDB(t, func(pool *pgxpool.Pool) {
        svc := service.NewItemService(pool, 20, 100)

        // Seed prerequisite data with raw SQL
        _, err := pool.Exec(ctx, `INSERT INTO users (...) VALUES (...)`)
        // ...

        // Test the service method
        result, err := svc.Create(ctx, &service.CreateItemInput{...})
        if err != nil { t.Fatalf("expected no error, got %v", err) }
        if result.Title != "Test Item" { t.Errorf("expected title 'Test Item', got %s", result.Title) }
    })
}
```

### Key Rules

- **Build tag**: `//go:build integration` at the top of the file.
- **File naming**: `x_integration_test.go` (not `x_test.go`) to make it clear.
- **Seed with raw SQL**: Use `pool.Exec` with INSERT statements. Never use service methods to seed — they have their own validation and side effects.
- **Test one thing per function**: Each test function should test one behavior (happy path, error case, permission check, etc.).
- **Clean state**: `WithTestDB` provides a fresh database for each test. No cleanup needed.

### What to Test

- **Happy paths**: Create, Read, Update, Delete with valid inputs.
- **Permission checks**: Wrong user type, non-owner, non-member. Expect `FORBIDDEN`.
- **Status guards**: Operation on wrong-status parent. Expect `BAD_REQUEST`.
- **Not found**: Non-existent ID. Expect `NOT_FOUND`.
- **Conflict**: Duplicate entries. Expect `CONFLICT`.
- **Edge cases**: Empty arrays, null optional fields, boundary values.

Reference: existing `_integration_test.go` files in `backend/internal/service/`

## Unit Tests (Pure Logic)

For validation helpers, parsing functions, and other pure logic — no build tag needed.

```go
package service

import "testing"

func TestNormalizePagination(t *testing.T) {
    tests := []struct {
        page, perPage   int
        wantP, wantPP   int
    }{
        {0, 0, 1, 20},
        {-1, 200, 1, 20},
        {5, 10, 5, 10},
    }
    for _, tt := range tests {
        p, pp := NormalizePagination(tt.page, tt.perPage, 20, 100)
        if p != tt.wantP || pp != tt.wantPP {
            t.Errorf("NormalizePagination(%d, %d) = (%d, %d), want (%d, %d)",
                tt.page, tt.perPage, p, pp, tt.wantP, tt.wantPP)
        }
    }
}
```

### Key Rules

- **No build tag** — runs on every `go test`.
- **Same package** — test unexported functions directly (e.g., `NormalizePagination`).
- **Table-driven tests** — use `[]struct` pattern for multiple cases.
- **File naming**: `x_test.go` in the same directory as the code.

Reference: existing `_test.go` files in `backend/internal/service/`

## Mock-based Handler Tests

Create a mock struct implementing the service interface. Set function fields for methods under test. Unimplemented methods panic. Inject mock directly into handler struct: `h := &AuthHandler{authService: &mockAuthService{...}}`. See `handler/auth_test.go` for examples.

## Scaffold-Generated Tests

`make new-module name=items` automatically generates `handler/item_test.go` with:

- A `mockItemService` struct implementing the `itemServicer` interface
- A `testItemDetail()` helper returning sample data
- 5 test stubs: Create success, Create validation error, List, GetByID not found, Delete

After scaffolding, fill in test bodies for your specific validation rules, error paths, and edge cases. The generated tests follow the same pattern as `auth_test.go`.

## Always Test Error Paths

Don't just test happy paths. Every service method should have tests for:

- **Permission denied** — wrong user type, non-owner, non-member → expect `FORBIDDEN`
- **Invalid status** — operation on wrong-status entity → expect `BAD_REQUEST`
- **Not found** — non-existent ID → expect `NOT_FOUND`
- **Duplicate/conflict** — re-creating existing entity → expect `CONFLICT`

These are the bugs that slip through manual testing and break in production.

## Component Unit Tests (SolidJS)

Use `@solidjs/testing-library` for component-level tests. Test the public API: renders, accepts props, fires events.

```tsx
import { render, screen, fireEvent } from "@solidjs/testing-library";
import { Button } from "./Button";

test("calls onClick", async () => {
  const handler = vi.fn();
  render(() => <Button onClick={handler}>Click</Button>);
  await fireEvent.click(screen.getByText("Click"));
  expect(handler).toHaveBeenCalledTimes(1);
});
```

Place test files next to the component: `Button.test.tsx` alongside `Button.tsx`. Vitest discovers `src/**/*.test.{ts,tsx}` automatically.

**Important:** `vitest.config.ts` must have `resolve.conditions: ["development", "browser"]` to prevent Solid from resolving server-side bundles in jsdom.

### Test Quality Bar

Every component test must assert at least one **behavioral property**, not just existence. "Renders without crashing" is not a test.

```tsx
// BAD — tests nothing meaningful

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [golid-ai/golid](https://github.com/golid-ai/golid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
