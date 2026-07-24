---
trigger: always_on
description: This document contains guidance for Claude (and other AI assistants) when working with the GeoIP2-java codebase. It captures architectural patterns, conventions, and lessons learned to help maintain consistency and quality.
---

# CLAUDE.md - GeoIP Java API

This document contains guidance for Claude (and other AI assistants) when working with the GeoIP2-java codebase. It captures architectural patterns, conventions, and lessons learned to help maintain consistency and quality.

## Project Overview

**GeoIP2-java** is MaxMind's official Java client library for:
- **GeoIP/GeoLite Web Services**: Country, City Plus, and Insights endpoints
- **GeoIP/GeoLite Databases**: Local MMDB file reading for various database types (City, Country, ASN, Anonymous IP, ISP, etc.)

The library provides both web service clients and database readers that return strongly-typed model objects containing geographic, ISP, anonymizer, and other IP-related data.

**Key Technologies:**
- Java 17+ (using modern Java features like records)
- Jackson for JSON serialization/deserialization
- MaxMind DB reader for binary database files
- Maven for build management
- JUnit 5 for testing
- WireMock for web service testing

## Code Architecture

### Package Structure

```
com.maxmind.geoip2/
├── model/              # Response models (CityResponse, InsightsResponse, etc.)
├── record/             # Data records (City, Location, Traits, Anonymizer, etc.)
├── exception/          # Custom exceptions for error handling
├── DatabaseReader      # Local MMDB file reader
├── WebServiceClient    # HTTP client for MaxMind web services
└── DatabaseProvider/WebServiceProvider interfaces
```

### Key Design Patterns

#### 1. **Java Records for Immutable Data Models**
All model and record classes use Java records for immutability and conciseness:

```java
public record Anonymizer(
    @JsonProperty("confidence")
    Integer confidence,

    @JsonProperty("is_anonymous")
    boolean isAnonymous,

    // ... more fields
) implements JsonSerializable {
    // Compact canonical constructor for defaults
    public Anonymizer {
        // Set defaults for null values
    }
}
```

**Key Points:**
- Records provide automatic `equals()`, `hashCode()`, `toString()`, and accessor methods
- Use `@JsonProperty` for JSON field mapping
- Use `@MaxMindDbParameter` for database field mapping
- Implement compact canonical constructors to set defaults for null values

#### 2. **Alphabetical Parameter Ordering**
Record parameters are **always** ordered alphabetically by field name. This maintains consistency across the codebase:

```java
public record InsightsResponse(
    Anonymizer anonymizer,  // A comes first
    City city,              // C comes next
    Continent continent,    // C (alphabetically after "city")
    // ... etc.
)
```

#### 3. **Deprecation Strategy**

When deprecating fields:

**For record parameters** (preferred for new deprecations):
```java
public record Traits(
    @Deprecated(since = "5.0.0", forRemoval = true)
    @JsonProperty("is_anonymous")
    boolean isAnonymous,
    // ...
)
```

This automatically marks the accessor method (`isAnonymous()`) as deprecated.

**For JavaBeans-style getters** (legacy code only):
```java
@Deprecated(since = "5.0.0", forRemoval = true)
public String getUserType() {
    return userType();
}
```

**Do NOT add deprecated getters for new fields** - they're only needed for backward compatibility with existing fields that had JavaBeans-style getters before the record migration.

#### 4. **Default Constructors for Record Classes**

All record classes in `src/main/java/com/maxmind/geoip2/record/` should provide a no-arg constructor that sets sensible defaults:

```java
public Anonymizer() {
    this(null, false, false, false, false, false, false, null, null);
}
```

- Nullable fields → `null`
- Boolean fields → `false`

**Note:** Model classes in `src/main/java/com/maxmind/geoip2/model/` do not require default constructors as they are typically constructed from API responses.

#### 5. **Web Service Only vs Database Records**

Some record classes are only used by web services and do **not** need MaxMind DB support:

**Web Service Only Records** (no `@MaxMindDbParameter` or `@MaxMindDbConstructor`):
- Records that are exclusive to web service responses (e.g., `Anonymizer` for Insights API)
- Only need `@JsonProperty` annotations for JSON deserialization
- Simpler implementation without database parsing logic

**Database-Supported Records** (need `@MaxMindDbParameter` and often `@MaxMindDbConstructor`):
- Records used by both web services and database files (e.g., `Traits`, `Location`, `City`)
- Need both `@JsonProperty` and `@MaxMindDbParameter` annotations
- May need `@MaxMindDbConstructor` for date parsing or other database-specific conversion

**How to Determine:**
- Check the JavaDoc - does it say "This is only available from the X web service"?
- Look at existing similar records in the `record/` package
- If in doubt, ask - adding unnecessary database support adds complexity

## Testing Conventions

### Test Structure

Tests are organized by model/class:
- `src/test/java/com/maxmind/geoip2/model/` - Response model tests
- `src/test/resources/test-data/` - JSON fixtures for tests

### JSON Test Fixtures

When adding new fields to responses:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxmind/GeoIP2-java](https://github.com/maxmind/GeoIP2-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
