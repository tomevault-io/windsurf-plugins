---
trigger: always_on
description: provides `newMapper()`, `mapperBuilder()`, shared POJOs, and assertion helpers).
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jackson XML dataformat module extends Jackson to serialize/deserialize XML using the STAX (Streaming API for XML) processing engine. The goal is to emulate JAXB "code-first" data binding while leveraging Jackson's rich type system and features. This is NOT a full JAXB clone or general-purpose XML toolkit.

**Key Principle:** Any XML written by this module MUST be readable by this module (guaranteed round-trip support).

**This branch is Jackson 3.x.** That means:

- Java package root is `tools.jackson.dataformat.xml` (NOT `com.fasterxml.jackson.dataformat.xml`).
  Only the `com.fasterxml.jackson.annotation` annotations package keeps the old root.
- Maven coordinates are `tools.jackson.dataformat:jackson-dataformat-xml`, parent
  `tools.jackson:jackson-base`.
- Mappers are **immutable**: configure via `XmlMapper.builder()`, never via
  `mapper.enable(...)` / `mapper.disable(...)` (those no longer exist).
- Format features moved out of the parser/generator classes into top-level enums
  `XmlReadFeature` and `XmlWriteFeature`, and several defaults changed (see below).
- `JacksonXmlModule` is now `XmlModule`.
- The module is a real JPMS module (`src/main/java/module-info.java`).

## Build & Test Commands

This is a Maven-based project. Use the Maven wrapper (`./mvnw`) for consistent builds.

### Basic Commands

```bash
# Clean and build
./mvnw clean install

# Run all tests
./mvnw test

# Run tests without building
./mvnw surefire:test

# Run a single test class
./mvnw test -Dtest=XmlMapperTest

# Run a single test method
./mvnw test -Dtest=XmlMapperTest#testSimpleSerialization

# Build without running tests
./mvnw clean install -DskipTests

# Generate code coverage report (JaCoCo)
./mvnw clean test
# Report available at: target/site/jacoco/index.html
```

### JDK Baseline

- Java **17** is the baseline for both main and test sources (inherited from
  `jackson-base`); CI builds on JDK 17, 21 and 24.
- Unlike 2.x, there is **no** separate `src/test-jdk17` source root and no JDK profile:
  Records and other JDK 17+ tests live in the regular test tree
  (`.../xml/records/`, `.../xml/jdk17/`, `.../xml/tofix/records/`).

### Working with Test Files

- Test sources: `src/test/java/tools/jackson/dataformat/xml/`
- **Test base class: `XmlTestUtil`** — extend this for new tests (JUnit 5 based;
  provides `newMapper()`, `mapperBuilder()`, shared POJOs, and assertion helpers).
  All test classes use it.
- `XmlTestBase` (the deprecated JUnit 3/4 base from 2.x) has been **removed**.
- Tests use JUnit 5 (`org.junit.jupiter.api.Test`, static `Assertions` imports) only;
  JUnit 4 is no longer on the test classpath.

## Repository Branch Structure

- `3.x` — active development branch for the next 3.x minor (currently `3.3.0-SNAPSHOT`)
- `3.2` — maintenance branch for `3.2.x` patch releases
- `3.1` — maintenance branch for `3.1.x` patch releases
- `3.0` — maintenance branch for `3.0.x` patch releases
- `2.x` — 2.x development branch (currently `2.23.0-SNAPSHOT`); `2.22` etc. are its
  maintenance branches

Fixes flow upward: patch branch → next minor → `3.x`. Target a PR at the oldest branch
the fix should ship in.

Version numbers are inherited from the `tools.jackson:jackson-base` parent POM.

## High-Level Architecture

### Core Components Flow

```
User Code
    ↓
XmlMapper (extends ObjectMapper) ← Primary API entry point
    ↓
XmlFactory (extends TextualTSFactory, a TokenStreamFactory) ← Creates parsers/generators
    ├→ FromXmlParser (XML → JSON token stream)
    │    ├─ XmlTokenStream (STAX abstraction layer)
    │    └─ XMLStreamReader (Woodstox)
    │
    └→ ToXmlGenerator (JSON token stream → XML)
         └─ XMLStreamWriter (Woodstox)
```

### Builder-Style Construction

Mapper and factory are configured **exclusively** through builders (3.x mappers are
immutable after construction):

```java
XmlMapper mapper = XmlMapper.builder()
    .enable(XmlWriteFeature.WRITE_XML_DECLARATION)
    .defaultUseWrapper(false)
    .build();
```

`XmlMapper.Builder` extends `MapperBuilder` and adds `defaultUseWrapper(boolean)`,
`nameForTextElement(String)`, and `enable`/`disable`/`configure` overloads for
`XmlReadFeature` / `XmlWriteFeature`. Builder state is saved in `XmlBuilderState`
(a `MapperBuilderState`) so `rebuild()` / JDK serialization round-trip correctly.

`XmlFactoryBuilder` configures the `XMLInputFactory`/`XMLOutputFactory`, the STAX
`ClassLoader`, the `XmlNameProcessor`, `nameForTextElement`, and format features.

Other entry points:

- `XmlMapper.shared()` — globally shared default-configured instance (handy for
  untyped/tree-model work). Do not mutate expectations around it; it is immutable.
- `XmlMapper.builderWithJackson2Defaults()` / `XmlMapper.Builder.configureForJackson2()`
  and `XmlFactory.builderWithJackson2Defaults()` — start from settings closer to
  Jackson 2.x defaults. Still a work in progress; it does not replicate 2.x exactly.

### Module Registration System

`XmlModule` (a `JacksonModule`) centralizes XML-specific configuration and is
registered automatically by `XmlMapper.Builder`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FasterXML/jackson-dataformat-xml](https://github.com/FasterXML/jackson-dataformat-xml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
