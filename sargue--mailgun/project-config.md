---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the Mailgun Java library codebase.
---

# CLAUDE.md - Mailgun Java Library

This document provides comprehensive guidance for AI assistants working with the Mailgun Java library codebase.

## Project Overview

**Project Name:** Mailgun Java Library
**Current Version:** 2.0.0
**License:** MIT
**Language:** Java (requires Java 8+)
**Build Tool:** Gradle 7.6
**Package:** `net.sargue:mailgun`

### What This Library Does

A small Java library for sending email messages using the Mailgun REST API service. It provides a fluent interface (DSL-style) for building and sending emails with support for:
- Simple text and HTML emails
- Attachments and inline images
- Templates with parameters
- Async sending
- Multipart messages
- HTML content helpers for basic formatting

### Important Context

1. **Official Mailgun Library Exists:** Mailgun now has an [official Java library](https://github.com/mailgun/mailgun-java). This library is maintained independently.
2. **Version 2.x Migration:** Version 2.x uses `jakarta.*` package naming (JakartaEE 9). Version 1.x uses legacy `javax.*` naming.
3. **Android Support:** Not officially supported, but can work with specific configuration (see README.md:98-106)

## Codebase Structure

```
mailgun/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── net/sargue/mailgun/
│   │           ├── Configuration.java          # Main configuration class (thread-safe, reusable)
│   │           ├── Mail.java                   # Abstract mail representation
│   │           ├── MailBuilder.java            # Fluent builder for emails
│   │           ├── MailForm.java               # Form-based mail implementation
│   │           ├── MailMultipart.java          # Multipart mail implementation
│   │           ├── MultipartBuilder.java       # Builder for multipart emails
│   │           ├── MailgunException.java       # Custom exception
│   │           ├── Response.java               # API response wrapper
│   │           ├── MailRequestCallback.java    # Async callback interface
│   │           ├── MailRequestCallbackFactory.java
│   │           ├── MailSendFilter.java         # Filter for conditional sending
│   │           ├── package-info.java
│   │           └── content/                    # HTML content generation helpers
│   │               ├── Body.java               # Body content builder
│   │               ├── Builder.java            # Generic builder interface
│   │               ├── MailContent.java        # Content representation
│   │               ├── MessageBuilder.java     # Message-level builder
│   │               ├── ContentConverter.java   # Extensible content conversion
│   │               ├── Util.java               # Utility functions
│   │               └── package-info.java
│   └── test/
│       └── java/
│           └── net/sargue/mailgun/test/
│               ├── BasicTests.java             # Core functionality tests
│               ├── ConfigurationTests.java     # Configuration tests
│               ├── ContentTests.java           # Content generation tests
│               ├── HTMLContentTests.java       # HTML content tests
│               └── ExampleTest.java            # Usage examples
├── build.gradle                                # Gradle build configuration
├── gradle.properties                           # Gradle properties (credentials)
├── settings.gradle                             # Gradle settings
├── gradlew                                     # Gradle wrapper (Unix)
├── gradlew.bat                                 # Gradle wrapper (Windows)
├── .travis.yml                                 # Travis CI configuration
├── .gitignore                                  # Git ignore rules
├── README.md                                   # User documentation
├── CHANGELOG.md                                # Version history
├── PUBLISH.md                                  # Publishing notes (Catalan)
└── LICENSE.txt                                 # MIT License
```

## Key Architecture Patterns

### 1. Fluent Interface (Builder Pattern)

The library is designed as a fluent API. Users chain method calls to build emails:

```java
Mail.using(configuration)
    .to("user@example.com")
    .subject("Test")
    .text("Hello!")
    .build()
    .send();
```

**Important:** When modifying builders, maintain this fluent pattern by returning `this` or the appropriate builder type.

### 2. Configuration Singleton

The `Configuration` class:
- Is designed to be built once and reused throughout the application
- Is **thread-safe**
- Contains a single JAX-RS client for all requests
- Must be closed when no longer needed to free resources
- Supports default parameters that apply to all emails

**When modifying:** Ensure thread safety is maintained for any new fields.

### 3. Abstract Mail with Concrete Implementations

- `Mail` is abstract with two implementations:
  - `MailForm`: Simple form-based emails
  - `MailMultipart`: Emails with attachments/inline images

### 4. Content Converter Extension System

The `ContentConverter` interface allows extending the library to handle custom objects as email content. Converters are registered with the `Configuration` object.

## Dependencies

### Runtime Dependencies
- **Jersey 3.0.9** (JAX-RS implementation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sargue/mailgun](https://github.com/sargue/mailgun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
