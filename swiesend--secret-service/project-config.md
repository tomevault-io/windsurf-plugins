---
trigger: always_on
description: This is the unified instruction file for all AI agents (Claude Code, GitHub Copilot, etc.) working on the `secret-service` codebase. For the full design roadmap and historical context, see [`docs/vision.md`](../docs/vision.md).
---

# Agent Instructions

This is the unified instruction file for all AI agents (Claude Code, GitHub Copilot, etc.) working on the `secret-service` codebase. For the full design roadmap and historical context, see [`docs/vision.md`](../docs/vision.md).

## Project Overview

**secret-service** is a Java library for storing secrets in a keyring over D-Bus, implementing the [freedesktop.org Secret Service API 0.2](https://specifications.freedesktop.org/secret-service/0.2/). It is the Java equivalent of the `libsecret` C library, compatible with GNOME Linux systems (gnome-keyring, KeePassXC).

- **Group/Artifact:** `de.swiesend:secret-service`
- **License:** MIT
- **JDK:** Requires JDK 17+ to build

## Build Commands

```bash
mvn clean compile          # Compile
mvn test                   # Run tests (requires D-Bus session + gnome-keyring)
mvn package                # Build JAR
mvn clean -Pcoverage test  # Run tests with JaCoCo coverage
```

Maven 3.6.0+ is enforced. No Gradle support.

## Project Structure

```
src/main/java/
  module-info.java                              # JPMS module: de.swiesend.secretservice
  de/swiesend/secretservice/
    functional/                                 # NEW FUNCTIONAL API (recommended)
      SecretService.java, Session.java, Collection.java, System.java
      interfaces/
        ServiceInterface.java, SessionInterface.java, CollectionInterface.java, SystemInterface.java
    simple/
      SimpleCollection.java                     # LEGACY API (backward-compatible adapter)
      interfaces/SimpleCollection.java
    interfaces/                                 # D-Bus interface definitions
      Service.java, Collection.java, Item.java, Session.java, Prompt.java
    handlers/
      MessageHandler.java, SignalHandler.java, Messaging.java
    errors/
      IsLocked.java, NoSession.java, NoSuchObject.java
    gnome/keyring/                              # GNOME-specific non-standard interfaces
      InternalUnsupportedGuiltRiddenInterface.java
    Service.java, Collection.java, Item.java    # Low-level API implementations
    Session.java, Prompt.java, Secret.java
    TransportEncryption.java                    # DH key exchange + AES-128-CBC
    Static.java                                 # Constants, conversion utilities

src/test/java/
  de/swiesend/secretservice/
    functional/                                 # Functional API tests
      SecretServiceTest.java, CollectionTest.java, SystemTest.java
      integration/Example.java
    integration/                                # Low-level API integration tests
      simple/SimpleCollectionTest.java
      ServiceTest.java, CollectionTest.java, ...
      test/Context.java                         # Shared test fixture helper
```

## Key Architecture Decisions

### Two API Layers

- **Functional API** (`de.swiesend.secretservice.functional`) — Recommended. Instance-scoped connections, `Optional` returns, `AutoCloseable` lifecycle. Entry point: `SecretService.create()`.
- **SimpleCollection** (`de.swiesend.secretservice.simple`) — Legacy backward-compatible adapter. Static shared D-Bus connection with JVM shutdown hook. Delegates internally to the functional API.

### Connection Ownership Model

- `System.connect()` creates an **owned** connection — `close()` disconnects it.
- `System.wrap(DBusConnection)` wraps an **existing** connection without ownership — `close()` is a no-op. Used by `SimpleCollection` to share its static connection with the functional layer.
- `SimpleCollection` wraps its static connection via `System.wrap()` so both layers share one D-Bus connection. The static `disconnect()` / shutdown hook owns the lifecycle.

### Secure Secret Cleanup

- `Secret` implements `AutoCloseable` — `close()` zeroes the `byte[] value` via `Arrays.fill`.
- The functional API never exposes `Secret` objects to callers. `getSecret()` returns `Optional<char[]>` after closing the `Secret` internally via try-with-resources.
- **`withSecret()` / `withSecrets()` callbacks** are the recommended API: the library decrypts, passes `char[]` to the callback, zeroes in `finally`. Callers never manage cleanup. The map passed to `withSecrets()` is unmodifiable, and values are snapshotted before the callback to prevent leak via map mutation.
- Avoid creating `new String(secret)` from `char[]` inside callbacks — `String` is immutable and cannot be cleared.

### D-Bus Message Handling

- `MessageHandler` constructs `MethodCall` messages via `connection.getMessageFactory().createMethodCall(...)`. The direct `new MethodCall(...)` constructor was made non-public in dbus-java 5.x; `MessageFactory` is the supported public alternative and works without reflection or `--add-opens`.
- dbus-java is pinned at 5.2.0. This resolves classpath collisions (e.g. issue #51) when other libraries on the classpath ship a newer dbus-java that no longer provides the 4.x public `MethodCall` constructor.
- Error responses are checked via `instanceof org.freedesktop.dbus.messages.Error` (the class moved from `org.freedesktop.dbus.errors.Error` in 4.x).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swiesend/secret-service](https://github.com/swiesend/secret-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
