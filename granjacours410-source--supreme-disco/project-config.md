---
trigger: always_on
description: Project Guidelines for Unit Testing
---


Reference: [MetaMask Unit Testing Guidelines](https://github.com/MetaMask/contributor-docs/blob/main/docs/testing/unit-testing.md)

# Unit Testing Guidelines

## Test Naming Rules

- **NEVER use "should" in test names** - this is a hard rule with zero exceptions
- **Use action-oriented descriptions** that describe what the code does
- **Be specific about the behavior being tested**
- **AVOID** weasel words like "handle", "manage", or other non-specific action verbs
- **AVOID** subjective outcome words like "successfully", "correctly", "invalid" - instead indicate what the actual result should be
- **BE SPECIFIC** about conditions: use "email without domain" instead of "invalid email"

```ts
// ❌ WRONG
it('should return fixed timestamp', () => { ... });
it('should ignore events', () => { ... });
it('should display error when input is invalid', () => { ... });
it('handles invalid input correctly', () => { ... });

// ✅ CORRECT
it('returns fixed timestamp for privacy events', () => { ... });
it('ignores events without privacy timestamp property', () => { ... });
it('displays error when email is missing @ symbol', () => { ... });
it('returns false for email without domain', () => { ... });
```

## Test Structure and Organization - MANDATORY

- **EVERY test MUST follow the AAA pattern** (Arrange, Act, Assert) with blank line separation
- **Each test must cover ONE behavior** and be isolated from others
- **Use helper functions** for test data creation
- **Group related tests** in `describe` blocks

```ts
it('returns false for email without domain', () => {
  const input = 'user@';

  const result = validateEmail(input);

  expect(result).toBe(false);
});
```

**Helper Functions**:
```ts
const createTestEvent = (overrides = {}) => ({
  type: EventType.TrackEvent,
  event: 'Test Event',
  timestamp: '2024-01-01T12:00:00.000Z',
  ...overrides,
});
```

## Mocking Rules - CRITICAL

- **EVERYTHING not under test MUST be mocked** - no exceptions
- **NO** use of `require` - use ES6 imports only
- **NO** use of `any` type - use proper TypeScript types
- **Mock all external dependencies** including APIs, services, hooks
- **Use realistic mock data** that reflects real usage

```ts
// ✅ CORRECT
import { apiService } from '../services/api';
jest.mock('../services/api');
const mockApiService = apiService as jest.Mocked<typeof apiService>;

interface MockEvent {
  type: EventType;
  event: string;
  timestamp: string;
}

// ❌ WRONG
const mockApi = require('../services/api'); // ❌ no require
const mockApi: any = jest.fn();             // ❌ no any type
```

## Test Isolation and Focus - MANDATORY

- **Each test MUST be independent** - no shared state between tests
- **Use `beforeEach` for setup, `afterEach` for cleanup**
- **Reset all mocks between tests**
- **Tests MUST run in any order**
- **Avoid duplicated or polluted tests**
- **Use mocks for all external dependencies**

```ts
// ✅ CORRECT Test Isolation
describe('MetaMetricsCustomTimestampPlugin', () => {
  let plugin: MetaMetricsCustomTimestampPlugin;

  beforeEach(() => {
    plugin = new MetaMetricsCustomTimestampPlugin({
      timestampStrategy: 'fixed',
      customTimestamp: '1970-01-01T00:00:00.000Z',
    });
    jest.clearAllMocks();
  });

  afterEach(() => {
    jest.resetAllMocks();
  });

  it('returns fixed timestamp for privacy events', () => {
    const event = createTestEvent({ privacyTimestamp: true });

    const result = plugin.execute(event);

    expect(result.timestamp).toBe('1970-01-01T00:00:00.000Z');
  });
});
```

## Test Coverage (MANDATORY)

**EVERY component MUST test:**
- ✅ **Happy path** - normal expected behavior
- ✅ **Edge cases** - null, undefined, empty values, boundary conditions
- ✅ **Error conditions** - invalid inputs, failure scenarios
- ✅ **Different code paths** - all if/else branches, switch cases
- ✅ **Method chaining** - for builder patterns
- ✅ **Side effects** - property changes, state updates, cleanup

```ts
// ✅ CORRECT Coverage Example
describe('MetaMetricsCustomTimestampPlugin', () => {
  describe('execute', () => {
    it('returns fixed timestamp for privacy events', () => {
      // Happy path
    });

    it('ignores events without privacy timestamp property', () => {
      // Edge case
    });

    it('throws error when strategy is null', () => {
      // Error condition
    });

    it('uses event-specific timestamp strategy when provided', () => {
      // Different code path
    });

    it('removes privacy properties from event', () => {
      // Side effect
    });
  });
});
```

## Parameterized Tests

- Parameterize tests to cover all values (e.g., enums) with type-safe iteration.

```ts
it.each(['small', 'medium', 'large'] as const)('renders %s size', (size) => {
  expect(renderComponent(size)).toBeOnTheScreen();
});
```

## Test Determinism

- **EVERYTHING** not under test must be mocked - no exceptions.
- Avoid brittle tests: do not test internal state or UI snapshots for logic.
- Only test public behavior, not implementation details.
- Mock time, randomness, and external systems to ensure consistent results.

```ts
// Mock all external dependencies
jest.mock('../services/api');
jest.mock('../utils/date');
jest.mock('../hooks/useAuth');

jest.useFakeTimers();
jest.setSystemTime(new Date('2024-01-01'));
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [granjacours410-source/supreme-disco](https://github.com/granjacours410-source/supreme-disco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
