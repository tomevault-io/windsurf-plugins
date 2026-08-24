---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Grafana Faro React Native SDK - a monorepo containing packages for real user monitoring (RUM) in React Native applications. The codebase was migrated from the faro-web-sdk repository with full git history preserved.

**Key Packages:**

- `@grafana/faro-react-native` - Core SDK with instrumentations, metas, and transports
- `@grafana/faro-react-native-tracing` - OpenTelemetry distributed tracing integration
- `@grafana/faro-test-utils` - Internal test utilities (not published)
- `demo/` - Full-featured demo application (not published)

**Critical Dependencies:**

- `@grafana/faro-core@^2.2.3` - Using caret versioning for minor/patch updates.
- React Native 0.82.1+
- Yarn Berry 4.12.0

## Build System & Commands

### Essential Commands

```bash
# Install dependencies (from root)
yarn install

# Build all packages
yarn build

# Run all tests
yarn quality:test

# Run tests for a single package
cd packages/react-native && yarn quality:test

# Run specific test file
cd packages/react-native && npx jest src/path/to/file.test.ts

# Run specific test case
cd packages/react-native && npx jest -t "test name pattern"

# Lint everything
yarn quality:lint

# Format everything
yarn quality:format

# Check for circular dependencies
yarn quality:circular-deps
```

### Demo App Commands

Commands to run from root folder:

```bash
# Start Metro bundler
yarn start:demo

# Run on iOS (from root)
yarn ios

# Run on Android (from root)
yarn android

```

### iOS Native Module Setup

```bash
cd demo/ios
pod install
cd ../..
yarn ios
```

## Architecture

### Monorepo Structure

This is a Lerna + Yarn Workspaces monorepo with critical hoisting configuration:

```json
"installConfig": {
  "hoistingLimits": "workspaces"
}
```

**Why this matters:** React and React Native MUST be hoisted to root `node_modules` to prevent Metro bundler issues. The `hoistingLimits: "workspaces"` setting ensures workspace dependencies stay within their packages while allowing shared deps to hoist.

### Package Build System

Each package builds to multiple formats:

- **CommonJS**: `dist/cjs/` - Node.js compatibility
- **ESM**: `dist/esm/` - Modern bundlers
- **TypeScript**: `dist/types/` - Type definitions

**Build Process:**

1. TypeScript compiles with separate configs: `tsconfig.cjs.json` and `tsconfig.esm.json`
2. Both extend from `tsconfig.base.{cjs,esm}.json` at root
3. Demo app uses source files directly via `react-native` field in package.json for fast development

### Instrumentation Architecture

All instrumentations extend `BaseInstrumentation` from `@grafana/faro-core`. The lifecycle is:

1. **Registration**: Added to config via `getRNInstrumentations()` or manually
2. **Initialization**: `initialize()` called by Faro core during `initializeFaro()`
3. **Runtime**: Instrumentations patch globals, register listeners, collect telemetry
4. **Cleanup**: `unpatch()` called on shutdown (if implemented)

**Key Pattern - Global Patching:**
Many instrumentations patch global objects (console, fetch, ErrorUtils). Always:

- Store original reference before patching
- Restore original in `unpatch()`
- Handle missing globals gracefully (not all RN versions have everything)

**Key Pattern - Avoiding Infinite Loops:**
Console/Error instrumentations must NOT use console.log or trigger errors internally. Use `this.logDebug()` / `this.logInfo()` from BaseInstrumentation which uses faro-core's internal logger.

### Transport System

The SDK uses a batched transport system:

- `FetchTransport` - Sends telemetry to Grafana Cloud collector
- `ConsoleTransport` - Logs telemetry to console for debugging
- Both implement circuit breaker pattern to handle offline scenarios

**Critical:** Transports must NOT be traced by HttpInstrumentation or TracingInstrumentation. Collector URLs are automatically filtered to prevent infinite loops.

### Testing Strategy

**Test Environment Configuration:**

- Base config: `jest.config.base.js` with shared moduleNameMapper
- Per-package: Each package has own `jest.config.js` that extends base
- Test utils: `@grafana/faro-test-utils` package provides mocks

**Critical Test Patterns:**

1. **Mock React Native APIs:**

```typescript
(AppState as any).addEventListener = jest.fn((event, handler) => ({
  remove: jest.fn(),
}));
```

2. **Use fake timers for time-based tests:**

```typescript
jest.useFakeTimers();
// ... test code
jest.runAllTimers();
jest.useRealTimers();
```

3. **Clean up between tests:**

```typescript
beforeEach(() => {
  jest.clearAllMocks();
  jest.clearAllTimers();
});
```

## Code Practices

### Defensive Null Checking

**DO NOT use non-null assertions (`!`)** - Use defensive null checking instead:

```typescript
// ❌ Bad
const value = match[2]!;

// ✅ Good
if (match && match[2]) {
  const value = match[2];
}
```

### Import Order

ESLint enforces alphabetical import order with group separation:

1. Built-in + external packages
2. `@grafana/*` packages (internal group)
3. Parent imports
4. Sibling imports

**Auto-fix:** `yarn quality:format` will fix import order.

### TypeScript Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/faro-react-native-sdk](https://github.com/grafana/faro-react-native-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
