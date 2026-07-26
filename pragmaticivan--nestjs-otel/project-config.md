---
trigger: always_on
description: This is a TypeScript library that provides OpenTelemetry integration for NestJS applications, enabling observability through metrics, tracing, and logging. The library follows NestJS module patterns and provides decorators and services for seamless OpenTelemetry integration.
---

# NestJS OpenTelemetry (nestjs-otel) - AI Coding Agent Instructions

## Project Overview

This is a TypeScript library that provides OpenTelemetry integration for NestJS applications, enabling observability through metrics, tracing, and logging. The library follows NestJS module patterns and provides decorators and services for seamless OpenTelemetry integration.

## Architecture & Key Components

### Module Structure
- **Entry point**: `src/index.ts` - exports all public APIs
- **Core module**: `src/opentelemetry-core.module.ts` - handles OpenTelemetry SDK integration and DI setup
- **Public module**: `src/opentelemetry.module.ts` - provides `forRoot()` and `forRootAsync()` methods
- **Services**: `TraceService` and `MetricService` - injectable services for manual instrumentation
- **Decorators**: `@Span`, `@OtelCounter`, `@OtelMethodCounter`, etc. - declarative instrumentation

### Configuration Pattern
```typescript
// Synchronous configuration
OpenTelemetryModule.forRoot({
  metrics: {
    hostMetrics: true,
  }
})

// Asynchronous configuration with factory
OpenTelemetryModule.forRootAsync({
  useFactory: () => ({ metrics: { hostMetrics: true } }),
  inject: [ConfigService]
})
```

### Decorator Patterns
- **Span decorator**: Auto-wraps methods in OpenTelemetry spans, supports both sync/async functions. Now supports `onResult` callback for capturing return values.
- **Traceable decorator**: Class-level decorator (`@Traceable`) that automatically applies `@Span` to all methods in a class.
- **Context decorators**: `@CurrentSpan` and `@Baggage` for accessing OpenTelemetry context (active span, baggage items) in NestJS controllers/resolvers.
- **Metric decorators**: Parameter injection decorators (`@OtelCounter`) and method/class decorators (`@OtelMethodCounter`, `@OtelInstanceCounter`)
- **Naming convention**: Auto-generated metric names follow `app.ClassName.methodName.calls.total` pattern

## Development Workflow

### Build & Test Commands
```bash
npm run build          # TypeScript compilation to /lib
npm run test          # Runs unit + e2e tests
npm run test:unit     # Jest unit tests
npm run test:e2e      # E2E tests with test applications
npm run test:coverage # Coverage report
npm run lint          # Prettier formatting check
npm run format        # Auto-format code
```

### File Structure Conventions
- **Source**: `src/` contains TypeScript source files
- **Compiled**: `lib/` contains compiled JavaScript (git-ignored, npm-published)
- **Tests**: Unit tests alongside source files (`.spec.ts`), E2E tests in `tests/e2e/`
- **Fixture app**: `tests/fixture-app/` provides test NestJS application for E2E testing

## Critical Implementation Patterns

### Decorator Implementation
When creating new decorators, follow the proxy pattern used in `@Span`:
```typescript
// Preserve function metadata and parameters for OpenAPI/reflection
propertyDescriptor.value = new Proxy(originalFunction, {
  apply: (_, thisArg, args) => wrappedFunction.apply(thisArg, args)
});
copyMetadataFromFunctionToFunction(originalFunction, propertyDescriptor.value);
```

### Metric Management
- Metrics are singleton instances managed by `metric-data.ts` using `getOrCreate*` functions
- Use the `OTEL_METER_NAME` constant for consistent meter naming
- Metrics support optional prefixes and default attributes from configuration

### Error Handling in Spans
Always record exceptions and set error status:
```typescript
span.recordException(error);
span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
```

### HTTP Adapter Detection
The library supports both Express and Fastify through `feature-detection.utils.ts`. When adding HTTP-related features, use `detectHttpAdapterType()` for adapter-specific logic.

## Dependencies & Integration Points

### Peer Dependencies
- `@opentelemetry/sdk-node` (required)
- `@opentelemetry/exporter-prometheus` (optional, for metrics)
- NestJS core packages for DI and decorators

### External Integrations
- **Prometheus**: Metrics exported via PrometheusExporter on port 8081
- **Jaeger/Tempo**: Tracing via BatchSpanProcessor
- **Pino Logging**: Structured logging with trace context injection

## Testing Strategy

### Unit Tests
- Test decorators with mock functions and verify OpenTelemetry API calls
- Test services in isolation with mocked OpenTelemetry providers
- Use `jest.mock()` for OpenTelemetry SDK components

### E2E Tests
- Create test NestJS applications in `tests/fixture-app/`
- Test module registration with both `forRoot()` and `forRootAsync()`
- Verify metrics/spans are created during actual HTTP requests
- Test adapter-specific behavior (Express vs Fastify)

## Common Patterns to Follow

1. **Export everything from index.ts** - All public APIs must be re-exported
2. **Use injection tokens** - Define constants in `opentelemetry.constants.ts`
3. **Global module pattern** - Core module is `@Global()` to avoid re-imports
4. **Async/sync support** - All decorators must handle both sync and async functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pragmaticivan/nestjs-otel](https://github.com/pragmaticivan/nestjs-otel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
