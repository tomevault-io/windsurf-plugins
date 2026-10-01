---
trigger: always_on
description: - This repository is Apache ShenYu, a high-performance, extensible, reactive API gateway for microservices.
---

# Apache ShenYu Copilot Development Instructions

## Project Context

- This repository is Apache ShenYu, a high-performance, extensible, reactive API gateway for microservices.
- The primary technology stack is Java 17, Spring Boot 3, Spring WebFlux/Reactor, and Maven. Most functionality is organized as a multi-module project.
- Use the Maven Wrapper included in the repository. On Windows, run `\.\mvnw.cmd`; on Linux/macOS, run `./mvnw`.
- Before making changes, read the target module's `pom.xml`, adjacent implementations, and tests. Existing conventions in the target module take precedence over general practices.

## Modules and Architecture

- `shenyu-web` and `shenyu-bootstrap` provide gateway runtime and startup integration.
- `shenyu-plugin` contains gateway plugins. Follow the existing plugin-chain, selector, rule, and handler patterns instead of bypassing the shared plugin API.
- `shenyu-admin` provides the administration service; `shenyu-sync-data-center` handles data synchronization between the admin service and gateway instances.
- `shenyu-client`, `shenyu-register-center`, and `shenyu-registry` provide service registration and discovery capabilities.
- `shenyu-common`, `shenyu-spi`, and `shenyu-infra` provide shared models, extension points, and infrastructure capabilities. Search for reusable implementations before adding shared logic; avoid duplication across modules.
- Preserve backward compatibility for SPIs, plugin names, selector/rule data structures, configuration keys, and serialization formats. Do not change public APIs or default behavior unless explicitly required.
- When adding a protocol, registry, data synchronization method, or plugin, follow the directory layout, wiring, SPI declarations, and configuration patterns of similar modules.

## Java Coding Standards

- Write code compatible with Java 17 and follow `script/shenyu_checkstyle.xml` and the existing style of the target module.
- Use lowercase package names, `PascalCase` class names, `camelCase` method and variable names, and `UPPER_SNAKE_CASE` constants.
- Do not use wildcard imports, `System.out.println`, or `printStackTrace()`. Use the project's established SLF4J logging patterns.
- Prefer immutable data, explicit generics, and existing project utilities. Do not use raw types, unchecked casts, or swallowed exceptions to bypass type safety or error handling.
- Catch only specific exceptions that can be handled. Include enough context in error logs to diagnose problems, but never log passwords, tokens, keys, or complete sensitive request data.
- Keep methods focused, and avoid deep nesting and duplicated logic. Do not introduce abstractions that duplicate existing architecture for a local requirement.
- Do not introduce blocking calls into reactive request chains. Avoid unnecessary `block()`, blocking I/O, or expensive work on event-loop threads. When blocking work is unavoidable, use the project's established scheduling and isolation patterns.
- Comments should explain design rationale, boundary conditions, or protocol details rather than restating the code. Use English Javadoc for public APIs and complex extension points, consistent with adjacent code. Do not add `@author` tags.
- Every new Java file must include the Apache License 2.0 header. Other checked files must follow the repository's existing license-header format.

## Configuration and Dependencies

- Follow the existing `shenyu.*` naming, binding classes, defaults, and documentation patterns for configuration. New configuration must consider default behavior and compatibility with existing properties.
- Manage dependency versions through properties or `dependencyManagement` in the root `pom.xml` whenever possible. Do not add arbitrary versions to child modules.
- Before adding a dependency, confirm that the repository does not already provide equivalent functionality. Consider its scope, transitive dependencies, license, and impact on release artifact size.
- Never commit credentials, private keys, access tokens, real internal addresses, or other sensitive data. Use clearly marked placeholder values in example configurations.

## Testing Standards

- Add or update tests when behavior changes. Place unit tests under the relevant module's `src/test/java` directory, using the `*Test.java` naming convention.
- Use the repository's existing JUnit 5, Mockito, Hamcrest/JUnit Assertions, and Reactor Test libraries. Prefer Reactor Test's `StepVerifier` for reactive flows.
- Cover normal paths, boundary conditions, invalid inputs, and regression scenarios. Keep tests deterministic and avoid real network calls, arbitrary time delays, and shared external services.
- For bug fixes, first add a regression test that reproduces the issue. Assert externally observable behavior rather than implementation details.
- Prefer testing the affected module and its dependencies first, for example:
  - Windows: `\.\mvnw.cmd -pl <module> -am test -DskipRemoteResources=true`
  - Linux/macOS: `./mvnw -pl <module> -am test -DskipRemoteResources=true`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/shenyu](https://github.com/apache/shenyu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
