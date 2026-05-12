---
trigger: always_on
description: Code style, testing patterns, and development workflows. For API reference and usage examples, see [llms.txt](./llms.txt).
---

# Contributor Guide for Midnight.js

Code style, testing patterns, and development workflows. For API reference and usage examples, see [llms.txt](./llms.txt).

## Code Style

### TypeScript

```typescript
// ✅ Use explicit types, avoid `any`
function processState(state: ContractState): ProcessedState { }

// ❌ Never use `any` or cast to `unknown`
function processState(state: any): any { }

// ✅ Use readonly for immutable data
interface Config {
  readonly networkId: string;
  readonly endpoints: readonly string[];
}

// ✅ Prefer union types over enums for simple cases
type TxStatus = 'pending' | 'confirmed' | 'failed';

```

### Naming Conventions

```typescript
// Interfaces: PascalCase, descriptive
interface PrivateStateProvider { }
interface DeployContractOptions { }

// Types: PascalCase
type ContractAddress = string;
type UnprovenTransaction = { /* ... */ };

// Functions: camelCase, verb-first
function deployContract() { }
function findDeployedContract() { }
function submitCallTx() { }

// Constants: SCREAMING_SNAKE_CASE
const DEFAULT_TIMEOUT_MS = 300000;
const MAX_RETRY_ATTEMPTS = 3;

// Files: kebab-case
// private-state-provider.ts
// deploy-contract.ts
```

## Architecture Patterns

### Provider Pattern

All capabilities are abstracted into pluggable providers. Use factory functions to create them; see [llms.txt](./llms.txt) for full API examples and `packages/types/src/` for interface definitions.

Key `PrivateStateProvider` methods: `set()`, `get()`, `remove()`, `clear()`, `setSigningKey()`, `exportPrivateStates()`, `importPrivateStates()`.

### Error Handling

```typescript
// ✅ Use custom error classes
class ContractDeploymentError extends Error {
  constructor(
    message: string,
    public readonly contractAddress: string,
    public readonly cause?: Error
  ) {
    super(message);
    this.name = 'ContractDeploymentError';
  }
}

// ✅ Preserve error chains
try {
  await deployContract(options);
} catch (error) {
  throw new ContractDeploymentError(
    'Failed to deploy contract',
    address,
    error instanceof Error ? error : undefined
  );
}

// ❌ Don't swallow errors silently
try {
  await riskyOperation();
} catch {
  // Silent failure - BAD
}
```

### Async Patterns

```typescript
// ✅ Use RxJS for streams and subscriptions
function contractStateObservable(address: string): Observable<ContractState> {
  return new Observable(subscriber => {
    const subscription = pollForChanges(address, state => {
      subscriber.next(state);
    });
    return () => subscription.unsubscribe();
  });
}

// ✅ Use async/await for sequential operations
async function deployAndCall(): Promise<Result> {
  const deployed = await deployContract(options);
  const result = await deployed.callTx.initialize();
  return result;
}
```

## Testing Requirements

### Test Structure

```typescript
// ✅ Use Arrange-Act-Assert pattern
describe('deployContract', () => {
  it('should deploy contract with initial state', async () => {
    // Arrange
    const providers = createMockProviders();
    const options = { compiledContract, privateStateId: 'test' };

    // Act
    const deployed = await deployContract(providers, options);

    // Assert
    expect(deployed.contractAddress).toBeDefined();
    expect(deployed.callTx).toBeDefined();
  });
});
```

### Test Categories

1. **Unit Tests** - Test individual functions in isolation
2. **Integration Tests** - Test provider interactions
3. **E2E Tests** - Test full transaction flows against network

### Testing Guidelines

```typescript
// ✅ Test meaningful scenarios, not implementation details
it('should encrypt state with provided password', async () => { });
it('should reject weak passwords', async () => { });

// ❌ Don't test internal methods or trivial getters
it('should call internal helper', async () => { }); // BAD

// ✅ Test error cases through behavior
it('should throw when contract address is invalid', async () => {
  await expect(findContract(invalidAddress))
    .rejects.toThrow(ContractNotFoundError);
});

// ✅ Use descriptive test names
it('should return undefined when private state does not exist', async () => { });
```

### Running Tests

```bash
# All tests
yarn test

# Specific package
yarn test --filter=@midnight-ntwrk/midnight-js-contracts

# Watch mode
yarn test --watch

# With coverage
yarn test --coverage
```

## Package-Specific Guidelines

### @midnight-ntwrk/midnight-js-types

- Define all shared interfaces here
- Export types that other packages depend on
- Keep type definitions minimal and focused

### @midnight-ntwrk/midnight-js-contracts

- High-level API for contract operations
- Functions should accept `MidnightProviders` as first argument
- Return well-typed results with transaction data

### @midnight-ntwrk/midnight-js-level-private-state-provider

- Security-critical: encryption, key derivation
- Use established crypto libraries only
- Never log sensitive data (passwords, keys, decrypted state)

### @midnight-ntwrk/midnight-js-indexer-public-data-provider

- GraphQL queries and subscriptions
- Handle network errors with retries
- Support both polling and real-time subscriptions

### @midnight-ntwrk/midnight-js-http-client-proof-provider


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midnightntwrk/midnight-js](https://github.com/midnightntwrk/midnight-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
