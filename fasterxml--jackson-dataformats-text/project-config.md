---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## General

Do not do "git commit" or "git push" commands without explicit approval.

## Project Overview

Jackson Dataformats Text is a multi-module umbrella project providing Jackson dataformat backends for text-based formats: CSV, Properties, YAML, and TOML. Each format module allows full Jackson API access (streaming, databinding, tree model) by extending core Jackson abstractions.

## Branch Strategy

- `3.x` - main branch, use for new features, pull requests.
- `3.1`, `3.2` - patch lines (`3.1.x`, `3.2.x`); bug fixes only, no new API.
- Use `@since 3.3` in Javadoc for new additions on the `3.x` branch: this must match
  the `3.x` pom version (currently 3.3.0-SNAPSHOT), so re-check it after a minor release.

Merges flow forward: `2.x` -> `3.1` -> `3.2` -> `3.x`. When a 2.x change is a new
feature, it must not land in the `3.1`/`3.2` patch lines: null-merge it at `3.1`
(`git merge -s ours 2.x`) so the merge is recorded but no content is taken, then port
it to `3.x` by hand. The null merge keeps future `2.x` -> `3.1` merges conflict-free.
Caution: `-s ours` discards *everything* pending from 2.x, not just the one change, so
verify `git log 3.1..2.x` holds only what you mean to drop.

## Build and Test Commands

This project uses Maven Wrapper (`./mvnw`). Requires Java 17+.

### Full Build and Test
```bash
./mvnw clean verify
```

### Build Without Tests
```bash
./mvnw clean install -DskipTests
```

### Run Tests Only
```bash
./mvnw test
```

### Run Tests for a Single Module
```bash
./mvnw test -pl csv                    # CSV module only
./mvnw test -pl properties             # Properties module only
./mvnw test -pl yaml                   # YAML module only
./mvnw test -pl toml                   # TOML module only
```

### Run a Single Test Class
```bash
# From module directory (e.g., csv/)
cd csv
../mvnw test -Dtest=CsvParserTest

# Or from root with module specification
./mvnw test -pl csv -Dtest=CsvParserTest
```

### Run Specific Test Method
```bash
./mvnw test -pl csv -Dtest=CsvParserTest#testSimpleExplicit
```

### Code Coverage
```bash
./mvnw test
# Coverage reports generated in target/site/jacoco/
```

## Architecture

### Module Structure

Each format module (csv, properties, yaml, toml) follows a consistent structure:

```
{module}/src/main/java/tools/jackson/dataformat/{format}/
├── {Format}Factory.java           # Entry point, extends TextualTSFactory
├── {Format}FactoryBuilder.java    # Builder pattern for factory configuration
├── {Format}Parser.java            # Reading/deserialization, extends ParserBase
├── {Format}Generator.java         # Writing/serialization, extends GeneratorBase
├── {Format}Mapper.java            # Convenience wrapper, extends ObjectMapper
├── {Format}Schema.java            # Format-specific metadata (if applicable)
├── {Format}ReadFeature.java       # Read configuration flags
├── {Format}WriteFeature.java      # Write configuration flags
├── impl/                          # Internal implementation (encoders, decoders)
└── util/ or io/                   # Helper utilities
```

### Core Extension Pattern

All formats extend Jackson core abstractions:

1. **Factory (TokenStreamFactory)**: Creates parsers and generators
   - Override `_createParser()` for various input sources
   - Override `_createGenerator()` / `_createUTF8Generator()` for output
   - Declare format capabilities via builder

2. **Parser (JsonParser)**: Token streaming for reading
   - Implement `nextToken()` state machine
   - Provide format-specific value accessors
   - Handle schema-based parsing where applicable

3. **Generator (JsonGenerator)**: Token streaming for writing
   - Implement `writeName()`, `writeString()`, `writeNumber()`, etc.
   - Handle nested structures per format rules
   - Support format-specific features

4. **Mapper (ObjectMapper)**: High-level databinding API
   - Convenience methods for schema generation from POJOs
   - Format-specific ObjectReader/ObjectWriter factories

### Schema Support

- **CSV**: Requires `CsvSchema` with column definitions, separators, quoting rules
- **Properties**: Uses `JavaPropsSchema` for path handling configuration
- **YAML/TOML**: Generally schema-less (inferred from data)

### Package Naming Convention

Jackson 3.x uses `tools.jackson.*` namespace (Jackson 2.x used `com.fasterxml.jackson.*`)

- Group ID: `tools.jackson.dataformat`
- Root package: `tools.jackson.dataformat.{format}`
- Internal packages: `tools.jackson.dataformat.{format}.impl`

## Testing

### Test Organization

Tests are organized by concern in each module:

```
{module}/src/test/java/tools/jackson/dataformat/{format}/
├── ModuleTestBase.java      # Shared test utilities and sample POJOs
├── deser/                   # Deserialization/parsing tests
├── ser/                     # Serialization/generation tests
├── schema/                  # Schema-related tests
├── filter/                  # Filtering/view tests
├── fuzz/                    # Fuzz testing
└── tofix/                   # Known issues awaiting fixes
```

### Test Framework


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FasterXML/jackson-dataformats-text](https://github.com/FasterXML/jackson-dataformats-text) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
