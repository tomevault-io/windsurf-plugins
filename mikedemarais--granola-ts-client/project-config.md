---
trigger: always_on
description: 1. **NEVER BREAK EXISTING USAGE PATTERNS**
---

# Development Guidelines

## CRITICAL: You Are Building a Library That Others Depend On

### Golden Rules of Library Development

1. **NEVER BREAK EXISTING USAGE PATTERNS**
   - If `new GranolaClient(token)` works today, it must work tomorrow
   - If it returns data today, it must return the same structure tomorrow
   - Breaking changes = major version bump (1.0.0 → 2.0.0)

2. **FAIL LOUDLY, NEVER SILENTLY**
   ```typescript
   // ❌ NEVER DO THIS:
   if (!token) return [];  // Silent failure
   
   // ✅ ALWAYS DO THIS:
   if (!token) throw new Error('Authentication required');
   ```

3. **TEST WITH REAL CONSUMER CODE**
   ```bash
   # Before ANY release:
   cd ../test-consumer-project
   npm link ../granola-ts-client
   npm test  # Must pass existing tests
   ```

## Library Architecture & Scope

### Core Principle: Separation of Concerns
- **granola-ts-client is a pure API client library** - it accepts tokens and makes API calls
- **Token extraction/management is NOT the library's responsibility** - that belongs to the application layer
- The library should remain **environment-agnostic** (work in browsers, Node.js, Bun, etc.)
- Do NOT add file system operations, token extraction, or environment-specific code to the library core

### What belongs in the library:
- API client methods (generated from OpenAPI spec)
- HTTP request handling and retry logic
- Type definitions for API responses
- Pagination utilities
- Error handling for API responses

### Optional Utilities (src/utils/):
- Token extraction utilities are exported separately via `granola-ts-client/utils`
- These are **optional helpers** that applications can choose to use
- They are Node.js/Bun specific and will not work in browsers
- The main client (`granola-ts-client`) does NOT depend on these utilities
- Applications can import only what they need:
  - Browser apps: `import GranolaClient from 'granola-ts-client'`
  - Node apps: Can also use `import { extractGranolaToken } from 'granola-ts-client/utils'`

### What does NOT belong in the library core:
- Direct usage of token extraction in the main client
- Authentication flows or token refresh logic in the main client
- Required dependencies on Node.js/Bun APIs
- Credential management in the main client

### Architecture Principle:
- Main client remains pure and environment-agnostic
- Optional utilities are clearly separated and opt-in only
- Build process compiles utilities with `--target node` separately
- Examples demonstrate both pure client usage and utility usage

## Backwards Compatibility Contract

### Constructor Compatibility Requirements

**The constructor is the most critical API surface. NEVER break it.**

When modifying constructor:
1. Support ALL previous signatures
2. Add new capabilities via overloading, not replacement
3. Test every historical usage pattern

```typescript
// If these work in version X, they must work in version X+1:
new GranolaClient('token')                    // v0.1.0 style
new GranolaClient({ apiKey: 'token' })       // v0.5.0 style  
new GranolaClient({ token: 'token' })        // v0.8.0 style
new GranolaClient()                           // v0.10.0 style
```

### Authentication State Management

**Authentication must be explicit and validated:**

```typescript
class GranolaClient {
  private assertAuthenticated(): void {
    if (!this.hasValidToken()) {
      throw new Error(
        'GranolaClient: No valid authentication token. ' +
        'Provide token via constructor or call setToken()'
      );
    }
  }
  
  async anyApiMethod() {
    this.assertAuthenticated();  // EVERY method must check
    // ... rest of implementation
  }
}
```

## API Design Principles

### Optional Parameters Are Dangerous

**Making required parameters optional is a BREAKING CHANGE:**

```typescript
// Version 1.0.0
constructor(token: string)  // Required

// Version 1.1.0 - THIS IS BREAKING!
constructor(token?: string)  // Optional

// Correct approach for 1.1.0:
constructor(token: string | Options)  // Overloaded
```

### Empty Data vs No Data vs Errors

**Be explicit about what empty responses mean:**

```typescript
interface ApiResponse<T> {
  data: T;           // Actual data
  hasData: boolean;  // Explicitly indicate emptiness
  error?: Error;     // Explicit errors
}

// Clear intention:
if (!response.hasData) {
  // Legitimately no data
}
if (response.error) {
  // Something went wrong
}
```

## Regression Prevention

### Required Integration Tests

**tests/client-compatibility.test.ts** must exist and test:

```typescript
describe('Backwards Compatibility', () => {
  // Test EVERY historical constructor pattern
  test('v0.3.0: new GranolaClient(token)', async () => {
    const client = new GranolaClient('test-token');
    expect(client).toBeDefined();
    // Mock and verify auth header is set
  });

  test('Fails clearly without auth', async () => {
    const client = new GranolaClient();
    await expect(client.getWorkspaces())
      .rejects.toThrow('Authentication required');
  });

  test('Returns expected data structure', async () => {
    const client = new GranolaClient('token');
    const transcript = await client.getDocumentTranscript('id');
    expect(Array.isArray(transcript)).toBe(true);
    // Never return undefined/null when authenticated
  });
});
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikedemarais/granola-ts-client](https://github.com/mikedemarais/granola-ts-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
