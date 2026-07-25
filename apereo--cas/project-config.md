---
trigger: always_on
description: This is the **Apereo CAS** (Central Authentication Service) codebase - a large-scale enterprise authentication server built with Java, Gradle, and Spring Boot.
---

# GitHub Copilot Instructions for Apereo CAS

This is the **Apereo CAS** (Central Authentication Service) codebase - a large-scale enterprise authentication server built with Java, Gradle, and Spring Boot.

> **Note**: If you're deploying/configuring CAS, use the **WAR Overlay** method, not this repository. Building from source is for contributors.

## Project Overview

- **Type**: Multi-module Gradle project (500+ modules)
- **Language**: Java 25+
- **Frameworks**: Spring Boot 3.x, Spring Cloud, Spring Webflow
- **Build Tool**: Gradle 8.x with parallel builds enabled
- **Architecture**: Modular design with clear separation between API, Core, Support, and Webapp layers

## Module Organization

```
api/           → Interface definitions and contracts (e.g., cas-server-core-api-authentication)
core/          → Core implementations (e.g., cas-server-core-authentication)
support/       → Feature modules (e.g., cas-server-support-ldap, cas-server-support-duo)
webapp/        → Web application modules (Tomcat, Jetty, etc.)
docs/          → Documentation and user guides
ci/            → CI scripts and test helpers
style/         → Checkstyle, SpotBugs, ErrorProne configs
```

**Key principle**: `api/` modules define contracts, `core/` implements them, `support/` adds features. Respect module boundaries.

## Build & Run

### Full Build (Skip Tests)
```bash
./gradlew build --parallel -x test -x javadoc -x check
```

### Build Specific Module
```bash
./gradlew :core:cas-server-core-authentication:build
```

### Run Locally
```bash
cd webapp/cas-server-webapp-tomcat
../../gradlew bootRun
```
Access at: `https://localhost:8443/cas`

### Clean Build
```bash
./gradlew clean build --no-build-cache
```

## Testing

### Test Framework
Use `./testcas.sh` for comprehensive testing:

```bash
# See available test categories
./gradlew -q testCategories

# Run specific test category
./testcas.sh --category CategoryName

# Run specific test class
./testcas.sh --test TestClassName

# Run with coverage
./testcas.sh --category CategoryName --with-coverage

# Debug mode
./testcas.sh --category CategoryName --debug
```

### Run Single Test (Direct Gradle)
```bash
# Run specific test class
./gradlew :core:cas-server-core-authentication:test --tests "*AuthenticationHandlerTests"

# Run specific test method
./gradlew :support:cas-server-support-ldap:test --tests "*LdapAuthenticationHandlerTests.verifySuccess"
```

### Test Conventions
- Every bug fix needs a test
- New features require comprehensive test coverage
- Tests belong in the same module as the code they test
- Use `@SpringBootTest` for integration tests
- Use `@Nested` for organizing related test cases

## Code Conventions

### Java Style
- **Java version**: Java 25+ (use modern features: records, pattern matching, switch expressions)
- **Indentation**: 4 spaces, NO tabs
- **Braces**: Always use braces, even for single-line blocks
- **Null safety**: Use `@NullMarked` at package level (JSpecify annotations)
- **Conditionals**: Avoid needless `else` statements
- **Imports**: No unused imports (enforced by Checkstyle)

### Lombok Usage
Lombok is heavily used. Key patterns:
- `@Getter` / `@Setter` for bean properties
- `@RequiredArgsConstructor` for dependency injection
- `@Slf4j` for logging (field name: `LOGGER`, static)
- `@ToString` / `@EqualsAndHashCode` with `doNotUseGetters = true`
- Avoid `@Data` (too implicit)

Example:
```java
@Slf4j
@RequiredArgsConstructor
public class MyService {
    private final ServiceManager servicesManager;
    
    public void doSomething() {
        LOGGER.debug("Processing request");
    }
}
```

### Spring Configuration
- Configuration classes use `@AutoConfiguration` or `@Configuration`
- Always use `@ConditionalOnFeatureEnabled` for feature toggles
- Use `@RefreshScope` for runtime-refreshable beans
- Use `@ConditionalOnMissingBean` to allow overrides
- Order beans with `@Order` or implement `Ordered`

Example:
```java
@AutoConfiguration
@EnableConfigurationProperties(CasConfigurationProperties.class)
@ConditionalOnFeatureEnabled(feature = CasFeatureModule.FeatureCatalog.LDAP)
public class LdapAuthenticationConfiguration {
    
    @Bean
    @RefreshScope(proxyMode = ScopedProxyMode.DEFAULT)
    @ConditionalOnMissingBean(name = "ldapAuthenticationHandler")
    public AuthenticationHandler ldapAuthenticationHandler(
        @Qualifier("ldapAuthenticationProperties") final LdapProperties ldapProperties) {
        // implementation
    }
}
```

### Module Dependencies
- `api/` modules: Define interfaces only, minimal dependencies
- `core/` modules: Depend on corresponding `api/` modules
- `support/` modules: Can depend on `core/` and other `support/` modules
- Avoid circular dependencies
- Use `api` vs `implementation` dependency scopes appropriately

### Package Structure
```java
// Always declare package with @NullMarked
@NullMarked
package org.apereo.cas.authentication;

import org.jspecify.annotations.NullMarked;
```

## Quality Checks

### Checkstyle
Enforced via `style/checkstyle-rules.xml`:
- Line length: 200 characters
- Suppressions in `style/checkstyle-suppressions.xml`

### SpotBugs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apereo/cas](https://github.com/apereo/cas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
