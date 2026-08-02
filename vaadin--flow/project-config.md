---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

For guidance on **designing a new feature** (API shape, browser-API wrapping,
signals, lifecycle, nullability, DOM events, bootstrap flow, Javadoc
expectations), see [DESIGN_GUIDELINES.md](DESIGN_GUIDELINES.md). This file
covers the operational side: repo structure, build commands, test workflow,
coding conventions, and general coding rules that apply to all changes.

## Repository Overview

Vaadin Flow is the Java framework of Vaadin Platform for building modern
web applications. This is a large, multi-module Maven project that combines
server-side Java components with modern frontend tooling (Vite, TypeScript,
React support).

### Key Architecture Components

**Core Server Framework (`flow-server/`)**:
- Component system with server-side state management (`StateNode`, `StateTree`)
- DOM abstraction layer (`Element`, `Node`) that syncs with client-side
- JavaScript execution bridge (`JacksonCodec`) for seamless
  client-server communication
- Routing system (`Router`, `RouteConfiguration`) with navigation lifecycle
- Dependency injection and instantiation (`Instantiator`, `Lookup`)
- Frontend asset management and bundling

**Client-Server Communication**:
- Uses Jackson for JSON serialization/deserialization between Java objects
  and JavaScript
- `executeJs()` methods allow calling JavaScript from server with automatic
  parameter serialization
- Return values from JavaScript can be automatically deserialized into
  Java beans
- WebSocket-based push communication (`PushConnection`,
  `AtmospherePushConnection`)

**Frontend Build System**:
- Vite-based development mode with hot reload
- Production bundling with webpack plugins
- TypeScript support with generated type definitions
- React and Lit template support
- Theme system with CSS custom properties

**Multi-Module Structure**:
- `flow-server`: Core server-side framework
- `flow-client`: Client-side TypeScript/JavaScript code
- `flow-data`: Data binding and validation
- `flow-router`: Navigation and routing
- `flow-html-components`: Basic HTML component wrappers
- `flow-tests/`: Extensive integration test suite
- `vaadin-spring`: Spring Framework integration

## Development Commands

### Building and Testing

```bash
# Build entire project
mvn clean install

# Build without tests (faster)
mvn clean install -DskipTests

# Note: To run tests, omit -DskipTests entirely (not -DskipTests=false)

# Build specific module
cd flow-server && mvn clean install

# Run tests for specific module
cd flow-server && mvn test

# Run specific test class
mvn test -Dtest=JacksonCodecTest

# Run specific test method
mvn test -Dtest=JacksonCodecTest#testComplexTypeSerialization

# Run tests matching pattern
mvn test -Dtest="*Codec*Test"

# Run integration tests (automatically starts and stops server)
cd flow-tests/test-root-context && mvn verify

# Run single integration test
mvn verify -Dit.test=ExecJavaScriptIT
```

### Code Quality

```bash
# Format code
mvn spotless:apply

# Check code formatting
mvn spotless:check

# Run checkstyle validation
mvn checkstyle:check
```

### Frontend Development

```bash
# Frontend assets are managed by Maven plugins
# Vite dev mode is automatically started for development
# Manual frontend build (rare, usually automatic):
cd flow-client && npm install && npm run build
```

## Coding Conventions

- Use triple quotes (`"""`) for multi-line string blocks in Java text blocks.
- **When tests fail, code doesn't compile, or similar issues occur: Always
  analyze why first. Do not start rewriting code.**
- **When writing code, names and comments should describe how the code works
  and why, not what has changed from previous versions. Commit messages
  capture change information, not the code itself.**
- **Always create proper tests for what should work first. If the tests
  expose problems in the implementation, fix the implementation after the
  tests have been created.**

## Working with Key Components

### JavaScript Execution and JSON Codec

The `JacksonCodec` class handles serialization between Java and
JavaScript:

- Parameters: Java objects → JSON → JavaScript variables (`$0`, `$1`, etc.)
- Return values: JavaScript objects → JSON → Java beans
- Special handling for `Element` instances (sent as DOM references)
- Support for arbitrary objects via Jackson serialization

When calling `executeJs()`, always pass values as parameters (`$0`, `$1`,
...) — never concatenate them into the expression string. See
[DESIGN_GUIDELINES.md](DESIGN_GUIDELINES.md) for the full rules.

### State Management

Flow uses a tree-based state management system:
- `StateNode`: Represents component state on server
- `StateTree`: Manages entire application state tree
- `NodeFeature`: Different aspects of node state (properties, children, etc.)
- Changes are automatically synchronized to client

### Component Development

Components extend `Component` and use:
- `Element`: Low-level DOM manipulation
- Property synchronization via `@Synchronize`
- Event handling with `@DomEvent`
- Client-side callbacks with `@ClientCallable`

### Testing

**Unit Tests**: Located in `src/test/java/` in each module
- Use JUnit 4/5

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vaadin/flow](https://github.com/vaadin/flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
