---
trigger: always_on
description: Javalin is a lightweight web framework for Java and Kotlin built on top of Jetty. This is a multi-module Maven project with Java 17+ support.
---

# Javalin Web Framework

Javalin is a lightweight web framework for Java and Kotlin built on top of Jetty. This is a multi-module Maven project with Java 17+ support.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Build
- **REQUIREMENT**: Always run `./mvnw package` before `./mvnw test` to avoid OSGI dependency errors
- Build without tests: `./mvnw package -DskipTests --batch-mode` -- takes 60 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
- Build with tests: `./mvnw package --batch-mode` -- takes 3+ minutes. NEVER CANCEL. Set timeout to 300+ seconds.
- **CRITICAL**: Browser tests fail in CI environment due to missing WebDriver - this is expected and normal
- Clean build: `./mvnw clean package -DskipTests --batch-mode` -- takes 90 seconds. NEVER CANCEL. Set timeout to 180+ seconds.
- Full CI build: `./mvnw -DRunningOnCi=true clean verify --batch-mode` -- takes 4+ minutes. NEVER CANCEL. Set timeout to 360+ seconds.

### Testing
- Run all tests: `./mvnw test --batch-mode` -- takes 2+ minutes. NEVER CANCEL. Set timeout to 240+ seconds.
- Run specific module tests: `./mvnw test -pl javalin --batch-mode`
- **EXPECTED FAILURES**: ~20 browser tests fail in CI environment (TestJavalinVueBrowser) due to WebDriver unavailability
- Non-browser tests pass reliably and provide good validation coverage

### Project Structure
- **Root**: Multi-module Maven project with 11 modules
- **Main module**: `javalin/` - Core framework implementation
- **Key modules**:
  - `javalin-testtools/` - Testing utilities
  - `javalin-rendering/` - Template engine plugins
  - `javalin-ssl/` - SSL/TLS helpers
  - `javalin-bundle/` - All-in-one bundle

## Validation Scenarios

### Always validate changes with these scenarios:

#### 1. Basic Server Functionality
```java
// Test script: Validate core Javalin functionality
var app = Javalin.create(config -> {
    // Basic config test
    config.routes.get("/", ctx -> ctx.result("Hello Javalin!"));
    config.routes.get("/health", ctx -> ctx.json(Map.of("status", "ok")));
}).start(7070);

// Test HTTP requests work
// Verify responses are correct
// Stop gracefully: app.stop()
```

#### 2. Build Validation Steps
1. `./mvnw clean` - Clean all build artifacts
2. `./mvnw package -DskipTests --batch-mode` - Verify compilation (60s)
3. Test basic server functionality with above scenario
4. `./mvnw test -pl javalin --batch-mode` - Run core tests (120s)

#### 3. Module Dependencies
- Always test the main `javalin` module can be used standalone
- Verify proper module separation (no circular dependencies)
- Check that examples in README.md work correctly

## Common Tasks

### Repository Navigation
```
javalin/
├── javalin/                 # Core framework (main module)
│   ├── src/main/java/io/javalin/
│   │   ├── Javalin.java    # Main entry point
│   │   ├── http/           # HTTP handling
│   │   ├── router/         # Routing implementation  
│   │   ├── config/         # Configuration
│   │   └── websocket/      # WebSocket support
│   └── src/test/java/      # Core tests
├── javalin-testtools/      # Testing utilities
├── javalin-rendering/      # Template engines
├── javalin-ssl/           # SSL helpers
└── .github/workflows/     # CI configuration
```

### Development Commands
- **Java version**: JDK 17+ required (project targets Java 17)
- **Maven wrapper**: `./mvnw` (no system Maven installation required)
- **Editor config**: `.editorconfig` - 4 spaces, UTF-8, LF line endings
- **Code style**: IntelliJ defaults with import optimization

### Commit Message Convention
Follow the repository's strict commit message format:
```
[component/area]: Description of change
```

**Examples:**
- `[core] Add feature for HTTP request handling`
- `[workflow] Update GitHub Actions dependencies`
- `[deps] Bump Jackson version to fix security issue`
- `[static-files] Fix path decoding for special characters`
- `[context] Add method to disable response compression`
- `[tests] Add unit tests for new validation logic`
- `[github] Update documentation and README files`

**Component categories commonly used:**
- `[core]` - Core framework functionality
- `[context]` - Request/response context changes
- `[router]` - Routing and handler logic
- `[static-files]` - Static file serving
- `[websocket]` - WebSocket functionality
- `[jetty]` - Jetty server configuration
- `[deps]` - Dependency updates
- `[tests]` - Test-related changes
- `[workflow]` - GitHub Actions and CI
- `[github]` - Documentation and repository files
- `[maven-release-plugin]` - Release process
- `[ssl]` - SSL/TLS functionality

### Testing Guidelines
- **Unit tests**: Fast, no external dependencies
- **Integration tests**: Use TestUtil.test() helper for server lifecycle
- **Browser tests**: Use WebDriverUtil (fails in CI, works locally with Chrome)
- **Test utilities**: io.javalin.testing package provides helpers

### Key Files to Monitor
- `pom.xml` - Main project configuration and dependencies
- `javalin/pom.xml` - Core module configuration  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [javalin/javalin](https://github.com/javalin/javalin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
