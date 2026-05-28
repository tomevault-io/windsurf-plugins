---
trigger: always_on
description: LangChain4J CDI is a Java library that integrates LangChain4J with CDI (Jakarta EE and Eclipse MicroProfile). It provides CDI extensions for building AI-powered applications using Java, Maven, and various application servers.
---

# LangChain4J CDI Integration
LangChain4J CDI is a Java library that integrates LangChain4J with CDI (Jakarta EE and Eclipse MicroProfile). It provides CDI extensions for building AI-powered applications using Java, Maven, and various application servers.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites
- Java 17 (OpenJDK Temurin recommended)
- Maven 3.9.11+ (use `./mvnw` wrapper included in the repository)
- Docker or Podman (for running examples with Ollama)

### Bootstrap and Build
Run these commands in sequence for first-time setup:

1. **Basic build (core modules only):**
   ```bash
   ./mvnw -B clean compile -DskipTests -pl '!examples/payara-car-booking'
   ```
   Takes ~1 minute. NEVER CANCEL. Set timeout to 5+ minutes.

2. **Full build with formatting validation:**
   ```bash
   ./mvnw -B formatter:validate verify -pl '!examples/payara-car-booking'
   ```
   Takes ~2 minutes. NEVER CANCEL. Set timeout to 10+ minutes.

3. **Install modules to local repository:**
   ```bash
   ./mvnw -B clean install -DskipTests -pl '!examples/payara-car-booking'
   ```
   Takes ~1 minute. NEVER CANCEL. Set timeout to 5+ minutes.

### Essential Commands

- **Format and validate code style:**
  ```bash
  ./mvnw -B -B spotless:check -pl '!examples/payara-car-booking'
  ```
  Takes ~30 seconds.

- **Format code (auto-fix):**
  ```bash
  ./mvnw -B spotless:apply -pl '!examples/payara-car-booking'
  ```
  Takes ~30 seconds.

- **Run unit tests:**
  ```bash
  ./mvnw -B clean test -pl '!examples/payara-car-booking,!langchain4j-cdi-integration-tests'
  ```
  Takes ~1.5 minutes. NEVER CANCEL. Set timeout to 8+ minutes.

- **Run all tests including integration:**
  ```bash
  ./mvnw -B clean test -pl '!examples/payara-car-booking'
  ```
  Takes ~2 minutes. NEVER CANCEL. Set timeout to 10+ minutes.

- **Check dependencies:**
  ```bash
  ./mvnw dependency:analyze -pl langchain4j-cdi-core
  ```

### Known Issues and Workarounds

- **Payara example fails** due to network connectivity to `nexus.dev.payara.fish`. Always exclude with `-pl '!examples/payara-car-booking'`.
- **CONTRIBUTING.md mentions `make lint` and `make format`** - these commands do NOT exist. Use Maven equivalents:
  - `make lint` → `./mvnw formatter:validate`
  - `make format` → `./mvnw formatter:format`
- **Spotless check fails** with "No such reference 'origin/main'" in fresh clones. Formatter validation is the primary linting mechanism.
- **Integration tests take longer** due to application server startup times.
- **Some Maven plugin versions missing** - warnings about missing versions for `jandex-maven-plugin` and `cargo-maven3-plugin` can be ignored.

## Running Examples

### Quarkus Car Booking Example

1. **Prerequisites:** Install all modules first (see Bootstrap section above)

2. **Start application:**
   ```bash
   cd examples/quarkus-car-booking
   ../../mvnw quarkus:dev
   ```
   Takes ~10 seconds to start. Application runs on http://localhost:8080

3. **Available endpoints:**
   - Chat: `GET /car-booking/chat?question=Hello%20how%20can%20you%20help%20me?`
   - Fraud detection: `GET /car-booking/fraud?name=Bond&surname=James`
   - OpenAPI spec: `GET /q/openapi`
   - Swagger UI: `GET /q/swagger-ui`

4. **Without Ollama:** Application starts but AI endpoints return connection errors. This is expected behavior.

5. **With Ollama:** Run the setup script first:
   ```bash
   ./runexample-quarkus.sh
   ```
   This script sets up Ollama with Docker/Podman and runs the application.

### Other Examples

All examples in `/examples/` directory follow similar patterns:
- **Helidon:** `examples/helidon-car-booking/`
- **GlassFish:** `examples/glassfish-car-booking/`
- **Liberty:** `examples/liberty-car-booking/`
- **Payara:** `examples/payara-car-booking/` (currently broken due to network issues)

Each has a `run.sh` script for execution.

## Validation

### Manual Testing Scenarios
After making changes, ALWAYS run through these validation steps:

1. **Basic build validation:**
   ```bash
   ./mvnw -B formatter:validate verify -pl '!examples/payara-car-booking'
   ```

2. **Integration test validation:**
   ```bash
   ./mvnw -B test -pl langchain4j-cdi-integration-tests/langchain4j-cdi-integration-tests-quarkus
   ```

3. **Example application validation:**
   ```bash
   cd examples/quarkus-car-booking
   ../../mvnw quarkus:dev
   # In another terminal:
   curl -s "http://localhost:8080/q/openapi" | grep -q "openapi"
   ```

### CI Validation Commands
These commands must pass before committing changes:

```bash
./mvnw -B formatter:validate verify -pl '!examples/payara-car-booking'
```

## Repository Structure

### Core Modules
- `langchain4j-cdi-core/` - Core CDI extension functionality
- `langchain4j-cdi-mp-config/` - Configuration support
- `langchain4j-cdi-portable-ext/` - Portable CDI extension
- `langchain4j-cdi-build-compatible-ext/` - Build-compatible CDI extension
- `langchain4j-cdi-fault-tolerance/` - Fault tolerance integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langchain4j/langchain4j-cdi](https://github.com/langchain4j/langchain4j-cdi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
