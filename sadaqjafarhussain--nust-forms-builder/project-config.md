---
trigger: always_on
description: From the **root directory** (formbricks/):
---

# Testing Patterns & Best Practices

## Running Tests

### Test Commands
From the **root directory** (formbricks/):
- `npm test` - Run all tests across all packages (recommended for CI/full testing)
- `npm run test:coverage` - Run all tests with coverage reports
- `npm run test:e2e` - Run end-to-end tests with Playwright

From the **apps/web directory** (apps/web/):
- `npm run test` - Run only web app tests (fastest for development)
- `npm run test:coverage` - Run web app tests with coverage
- `npm run test -- <file-pattern>` - Run specific test files

### Examples
```bash
# Run all tests from root (takes ~3 minutes, runs 790 test files with 5334+ tests)
npm test

# Run specific test file from apps/web (fastest for development)
npm run test -- modules/cache/lib/service.test.ts

# Run tests matching pattern from apps/web  
npm run test -- modules/ee/license-check/lib/license.test.ts

# Run with coverage from root
npm run test:coverage

# Run specific test with watch mode from apps/web (for development)
npm run test -- --watch modules/cache/lib/service.test.ts

# Run tests for a specific directory from apps/web
npm run test -- modules/cache/
```

### Performance Tips
- **For development**: Use `apps/web` directory commands to run only web app tests
- **For CI/validation**: Use root directory commands to run all packages
- **For specific features**: Use file patterns to target specific test files
- **For debugging**: Use `--watch` mode for continuous testing during development

### Test File Organization
- Place test files in the **same directory** as the source file
- Use `.test.ts` for utility/service tests (Node environment)
- Use `.test.tsx` for React component tests (jsdom environment)

## Test File Naming & Environment

### File Extensions
- Use `.test.tsx` for React component/hook tests (runs in jsdom environment)
- Use `.test.ts` for utility/service tests (runs in Node environment)
- The vitest config uses `environmentMatchGlobs` to automatically set jsdom for `.tsx` files

### Test Structure
```typescript
// Import the mocked functions first
import { useHook } from "@/path/to/hook";
import { serviceFunction } from "@/path/to/service";
import { renderHook, waitFor } from "@testing-library/react";
import { beforeEach, describe, expect, test, vi } from "vitest";

// Mock dependencies
vi.mock("@/path/to/hook", () => ({
  useHook: vi.fn(),
}));

describe("ComponentName", () => {
  beforeEach(() => {
    vi.clearAllMocks();
    // Setup default mocks
  });

  test("descriptive test name", async () => {
    // Test implementation
  });
});
```

## React Hook Testing

### Context Mocking
When testing hooks that use React Context:
```typescript
vi.mocked(useResponseFilter).mockReturnValue({
  selectedFilter: {
    filter: [],
    responseStatus: "all",
  },
  setSelectedFilter: vi.fn(),
  selectedOptions: {
    questionOptions: [],
    questionFilterOptions: [],
  },
  setSelectedOptions: vi.fn(),
  dateRange: { from: new Date(), to: new Date() },
  setDateRange: vi.fn(),
  resetState: vi.fn(),
});
```

### Testing Async Hooks
- Always use `waitFor` for async operations
- Test both loading and completed states
- Verify API calls with correct parameters

```typescript
test("fetches data on mount", async () => {
  const { result } = renderHook(() => useHook());
  
  expect(result.current.isLoading).toBe(true);
  
  await waitFor(() => {
    expect(result.current.isLoading).toBe(false);
  });
  
  expect(result.current.data).toBe(expectedData);
  expect(vi.mocked(apiCall)).toHaveBeenCalledWith(expectedParams);
});
```

### Testing Hook Dependencies
To test useEffect dependencies, ensure mocks return different values:
```typescript
// First render
mockGetFormattedFilters.mockReturnValue(mockFilters);

// Change dependency and trigger re-render
const newMockFilters = { ...mockFilters, finished: true };
mockGetFormattedFilters.mockReturnValue(newMockFilters);
rerender();
```

## Performance Testing

### Race Condition Testing
Test AbortController implementation:
```typescript
test("cancels previous request when new request is made", async () => {
  let resolveFirst: (value: any) => void;
  let resolveSecond: (value: any) => void;

  const firstPromise = new Promise((resolve) => {
    resolveFirst = resolve;
  });
  const secondPromise = new Promise((resolve) => {
    resolveSecond = resolve;
  });

  vi.mocked(apiCall)
    .mockReturnValueOnce(firstPromise as any)
    .mockReturnValueOnce(secondPromise as any);

  const { result } = renderHook(() => useHook());
  
  // Trigger second request
  result.current.refetch();
  
  // Resolve in order - first should be cancelled
  resolveFirst!({ data: 100 });
  resolveSecond!({ data: 200 });

  await waitFor(() => {
    expect(result.current.isLoading).toBe(false);
  });

  // Should have result from second request
  expect(result.current.data).toBe(200);
});
```

### Cleanup Testing
```typescript
test("cleans up on unmount", () => {
  const abortSpy = vi.spyOn(AbortController.prototype, "abort");
  
  const { unmount } = renderHook(() => useHook());
  unmount();
  
  expect(abortSpy).toHaveBeenCalled();
  abortSpy.mockRestore();
});
```

## Error Handling Testing

### API Error Testing
```typescript
test("handles API errors gracefully", async () => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SadaqJafarHussain/NUST-Forms-Builder](https://github.com/SadaqJafarHussain/NUST-Forms-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
