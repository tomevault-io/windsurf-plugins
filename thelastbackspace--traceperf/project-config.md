---
trigger: always_on
description: ├── src/                      # Source code
---

# TracePerf Development Guidelines

## 📂 Project Structure

```
traceperf/
├── src/                      # Source code
│   ├── index.ts              # Main entry point
│   ├── core/                 # Core functionality
│   │   ├── logger.ts         # Base logger implementation
│   │   ├── config.ts         # Configuration management
│   │   └── constants.ts      # Shared constants
│   ├── trackers/             # Performance tracking modules
│   │   ├── execution.ts      # Execution flow tracking
│   │   ├── performance.ts    # Performance monitoring
│   │   └── memory.ts         # Memory usage tracking
│   ├── formatters/           # Output formatting
│   │   ├── cli.ts            # CLI output formatting
│   │   ├── json.ts           # JSON output formatting
│   │   └── ascii.ts          # ASCII art generation
│   ├── utils/                # Utility functions
│   │   ├── timing.ts         # Timing utilities
│   │   ├── stack.ts          # Call stack utilities
│   │   └── colors.ts         # Terminal color utilities
│   └── types/                # TypeScript type definitions
│       ├── index.ts          # Type exports
│       ├── logger.ts         # Logger types
│       └── config.ts         # Configuration types
├── test/                     # Test files
│   ├── unit/                 # Unit tests
│   ├── integration/          # Integration tests
│   └── fixtures/             # Test fixtures
├── examples/                 # Example usage
│   ├── basic-logging.js      # Basic logging examples
│   ├── execution-flow.js     # Execution flow examples
│   └── performance.js        # Performance tracking examples
├── docs/                     # Documentation
│   ├── api/                  # API documentation
│   ├── guides/               # Usage guides
│   └── examples/             # Example documentation
├── scripts/                  # Build and utility scripts
├── .github/                  # GitHub configuration
│   └── workflows/            # GitHub Actions workflows
├── .eslintrc.js              # ESLint configuration
├── .prettierrc               # Prettier configuration
├── jest.config.js            # Jest configuration
├── tsconfig.json             # TypeScript configuration
├── package.json              # Package manifest
└── README.md                 # Project README
```

## 🧩 Code Architecture

### Core Principles

1. **Modularity**: Each component should have a single responsibility
2. **Extensibility**: Design for extension with plugins and custom formatters
3. **Performance**: Minimize overhead, especially in production environments
4. **Type Safety**: Use TypeScript for all code to ensure type safety
5. **Testing**: Maintain high test coverage for all components

### Design Patterns

1. **Singleton**: Use for the main logger instance
2. **Factory**: For creating different types of loggers and formatters
3. **Observer**: For event-based logging and notifications
4. **Decorator**: For adding functionality to loggers
5. **Strategy**: For different logging strategies based on environment

## 💻 Coding Standards

### General Guidelines

1. Use TypeScript for all code
2. Follow functional programming principles where appropriate
3. Minimize side effects
4. Use immutable data structures when possible
5. Avoid global state except for the main logger instance
6. Use async/await for asynchronous operations
7. Implement proper error handling throughout

### Naming Conventions

1. **Files**: Use kebab-case for filenames (e.g., `execution-tracker.ts`)
2. **Classes**: Use PascalCase for class names (e.g., `ExecutionTracker`)
3. **Functions/Methods**: Use camelCase for functions (e.g., `trackExecution()`)
4. **Constants**: Use UPPER_SNAKE_CASE for constants (e.g., `DEFAULT_TIMEOUT`)
5. **Interfaces/Types**: Use PascalCase with prefix I for interfaces (e.g., `ILoggerConfig`)
6. **Private Properties**: Use underscore prefix for private properties (e.g., `_config`)

### Code Style

1. Use 2 spaces for indentation
2. Maximum line length of 100 characters
3. Use semicolons at the end of statements
4. Use single quotes for strings
5. Always use curly braces for control structures
6. Add trailing commas in multi-line object/array literals
7. Use explicit type annotations for function parameters and return types

## 🚀 Performance Optimization

### General Optimizations

1. **Lazy Initialization**: Initialize components only when needed
2. **Caching**: Cache expensive operations and computed values
3. **Batching**: Batch log operations when possible
4. **Sampling**: Implement sampling for high-volume logs
5. **Async Logging**: Use non-blocking operations for I/O

### Production Optimizations

1. **Tree Shaking**: Ensure the library is tree-shakeable
2. **Dead Code Elimination**: Remove unused code in production builds
3. **Conditional Compilation**: Use environment flags to remove debug code
4. **Minimal Dependencies**: Minimize external dependencies
5. **Bundle Size**: Keep the bundle size as small as possible

## 📝 Documentation Standards

### Code Documentation

1. Use JSDoc comments for all public APIs
2. Include examples in documentation
3. Document parameters, return values, and exceptions
4. Add inline comments for complex logic
5. Keep comments up-to-date with code changes

### Example JSDoc Format

```typescript
/**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thelastbackspace/traceperf](https://github.com/thelastbackspace/traceperf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
