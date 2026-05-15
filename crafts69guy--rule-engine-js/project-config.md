---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

- **Version**: 1.0.7
- **Status**: Stable release with typed path autocomplete
- **Test Coverage**: 615 tests (614 passing, 1 skipped), >90% code coverage
- **Main Branch**: production

## Development Commands

### Build & Development

- `npm run dev` - Start development mode with watch compilation
- `npm run build` - Full production build (clean + lib + types + analyze)
- `npm run build:lib` - Build library bundles only
- `npm run build:types` - Generate TypeScript declarations
- `npm run clean` - Remove all generated files (dist/, types/, coverage/)

### Testing

- `npm test` - Run all tests with Jest
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage report
- `npm run test:all-envs` - Full test suite across Node versions and import formats
- `npm run test:quick` - Fast test run (used in pre-commit)

### Code Quality

- `npm run lint` - ESLint check on src/
- `npm run lint:fix` - Auto-fix ESLint issues
- `npm run lint:all` - Lint both src/ and tests/
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check formatting without modifying
- `npm run quality:check` - Comprehensive quality validation

### Release & Publishing

- `npm run release:patch` - Bump patch version and publish
- `npm run release:minor` - Bump minor version and publish
- `npm run release:major` - Bump major version and publish

## Architecture Overview

### Core Components

**RuleEngine** (`src/core/RuleEngine.js`)

- Main evaluation engine with LRU caching and performance metrics
- Manages operator registry and expression validation
- Provides security protections against prototype pollution
- Key methods: `evaluateExpr()`, `registerOperator()`, `getMetrics()`

**PathResolver** (`src/core/PathResolver.js`)

- Safe path resolution with dot notation support
- Caching for performance optimization
- Security measures against malicious paths
- Resolves both literal values and object paths

**StatefulRuleEngine** (`src/core/StatefulRuleEngine.js`)

- Wraps base RuleEngine to add state tracking and event-driven capabilities
- Maintains previous states for comparison across evaluations
- Event system: `triggered`, `untriggered`, `changed`, `evaluated`
- Optional evaluation history storage with configurable size limits
- Batch evaluation support via `evaluateBatch()`
- Flexible triggering modes (default: false → true transitions, optional: every change)
- Pure change rule detection for optimized triggering logic

**Phase 1 Enhancements (Production-Ready):**

- **State TTL/Expiration**: Automatic cleanup of stale states with configurable expiration times
- **Deep Copy Protection**: Prevents context mutation issues with circular reference handling
- **Listener Management**: Memory leak detection with configurable thresholds and cleanup methods
- **State Statistics**: Real-time monitoring of memory usage, rule counts, and state age

**Phase 2 Enhancements (Production-Ready):**

- **History Management Strategy Pattern**:
  - `GlobalHistoryManager`: FIFO queue with global size limit (legacy mode)
  - `PerRuleHistoryManager`: Separate queues per rule to prevent history domination
  - Configurable via `maxHistorySize` (global) or `maxHistoryPerRule` options
- **Batch Evaluation Error Handling**:
  - `stopOnError`: Option to halt batch processing on first error
  - `collectErrors`: Option to gather detailed error information for debugging
  - Returns structured response: `{ results, success, successCount, errorCount, totalCount, errors }`
  - Handles both engine-returned errors (`result.error`) and unexpected exceptions
- **Resource Cleanup**: `destroy()` method for proper resource management

**Phase 3.2 Concurrency Control (Production-Ready):**

- **ConcurrencyManager** (`src/core/concurrency/ConcurrencyManager.js`):
  - Per-rule execution queues with configurable concurrency limits
  - Automatic timeout handling with cleanup and callbacks
  - Queue management with max size limits and overflow callbacks
  - Promise-based async execution with proper error propagation
  - Statistics tracking: active, queued, completed, timeout counts
- **Integration**: All evaluation methods are now async (breaking change from Phase 2)
- **Configuration**: `concurrency` option with `maxConcurrent`, `timeout`, `onTimeout`, `onQueueFull`

**Phase 3.3 Enhanced Error Recovery (Production-Ready):**

- **RetryManager** (`src/core/recovery/RetryStrategies.js`):
  - Three retry strategies: exponential backoff, fixed delay, linear backoff
  - Configurable max attempts, delays, and retryable error patterns
  - Retry history tracking per rule with attempt timestamps
  - Optional retry callback hooks
- **CircuitBreaker** (`src/core/recovery/CircuitBreaker.js`):
  - State machine: closed (normal), open (failing), half-open (testing)
  - Configurable failure threshold and reset timeout
  - Automatic state transitions with timer management
  - Per-rule circuit tracking with statistics
  - Optional circuit state change callbacks
- **FallbackManager** (`src/core/recovery/FallbackManager.js`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crafts69guy/rule-engine-js](https://github.com/crafts69guy/rule-engine-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
