---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GeoIP2-php** is MaxMind's official PHP client library for:
- **GeoIP/GeoLite Web Services**: Country, City Plus, and Insights endpoints
- **GeoIP/GeoLite Databases**: Local MMDB file reading for various database types (City, Country, ASN, Anonymous IP, Anonymous Plus, ISP, etc.)

The library provides both web service clients and database readers that return strongly-typed model objects containing geographic, ISP, anonymizer, and other IP-related data.

**Key Technologies:**
- PHP 8.1+ (uses modern PHP features like readonly properties and strict types)
- MaxMind DB Reader for binary database files
- MaxMind Web Service Common for HTTP client functionality
- PHPUnit for testing
- php-cs-fixer, phpcs, and phpstan for code quality

## Code Architecture

### Package Structure

```
GeoIp2/
├── Model/              # Response models (City, Insights, AnonymousIp, etc.)
├── Record/             # Data records (City, Location, Traits, etc.)
├── Exception/          # Custom exceptions for error handling
├── Database/Reader     # Local MMDB file reader
├── WebService/Client   # HTTP client for MaxMind web services
└── ProviderInterface   # Common interface for database and web service
```

### Key Design Patterns

#### 1. **Readonly Properties for Immutable Data**
All model and record classes use PHP 8.1+ `readonly` properties for immutability and performance:

```php
class AnonymousPlus extends AnonymousIp
{
    public readonly ?int $anonymizerConfidence;
    public readonly ?string $networkLastSeen;
    public readonly ?string $providerName;
}
```

**Key Points:**
- Properties are set in the constructor and cannot be modified afterward
- Use `readonly` keyword for all public properties
- Nullable properties use `?Type` syntax
- Non-nullable booleans typically default to `false` in constructor logic

#### 2. **Inheritance Hierarchies**

Models follow clear inheritance patterns:
- `Country` → base model with country/continent data
- `City` extends `Country` → adds city, location, postal, subdivisions
- `Insights` extends `City` → adds additional web service fields
- `Enterprise` extends `City` → adds enterprise-specific fields

Records have similar patterns:
- `AbstractNamedRecord` → base with names/locales
- `AbstractPlaceRecord` extends `AbstractNamedRecord` → adds confidence, geonameId
- Specific records (`City`, `Country`, etc.) extend these abstracts

#### 3. **JsonSerializable Implementation**

All model and record classes implement `\JsonSerializable` for consistent JSON output:

```php
public function jsonSerialize(): ?array
{
    $js = parent::jsonSerialize();

    if ($this->anonymizerConfidence !== null) {
        $js['anonymizer_confidence'] = $this->anonymizerConfidence;
    }

    return $js;
}
```

- Only include non-null values in JSON output
- Use snake_case for JSON keys (matching API format)
- Properties use camelCase in PHP

#### 4. **Constructor Array Parameter Pattern**

Models and records are constructed from associative arrays (from JSON/DB):

```php
public function __construct(array $raw)
{
    parent::__construct($raw);
    $this->anonymizerConfidence = $raw['anonymizer_confidence'] ?? null;
    $this->networkLastSeen = $raw['network_last_seen'] ?? null;
}
```

- Use `$raw['snake_case_key'] ?? null` pattern for optional fields
- Use `$raw['snake_case_key'] ?? false` for boolean fields
- Call parent constructor first if extending another class

#### 5. **Web Service Only vs Database Models**

Some models are only used by web services and do **not** need MaxMind DB support:

**Web Service Only Models**:
- Models that are exclusive to web service responses
- Simpler implementation without database parsing logic
- Example: `Insights` (extends City but used only for web service)

**Database-Supported Models**:
- Models used by both web services and database files
- Must handle MaxMind DB format data structures
- Example: `City`, `Country`, `AnonymousIp`, `AnonymousPlus`

## Testing Conventions

### Running Tests

```bash
# Install dependencies
composer install

# Run all tests
vendor/bin/phpunit

# Run specific test class
vendor/bin/phpunit tests/GeoIp2/Test/Model/InsightsTest.php

# Run with coverage (if xdebug installed)
vendor/bin/phpunit --coverage-html coverage/
```

### Linting and Static Analysis

```bash
# PHP-CS-Fixer (code style)
vendor/bin/php-cs-fixer fix --verbose --diff --dry-run

# Apply fixes
vendor/bin/php-cs-fixer fix

# PHPCS (PSR-2 compliance)
vendor/bin/phpcs --standard=PSR2 src/

# PHPStan (static analysis)
vendor/bin/phpstan analyze

# Validate composer.json
composer validate
```

### Test Structure

Tests are organized by model/class:
- `tests/GeoIp2/Test/Database/` - Database reader tests
- `tests/GeoIp2/Test/Model/` - Response model tests
- `tests/GeoIp2/Test/WebService/` - Web service client tests

### Test Patterns

When adding new fields to models:
1. Update the test method to include the new field in the `$raw` array
2. Add assertions to verify the field is properly populated
3. Test both presence and absence of the field (null handling)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxmind/GeoIP2-php](https://github.com/maxmind/GeoIP2-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
