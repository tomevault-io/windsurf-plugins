---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GeoIP2-ruby** is MaxMind's official Ruby client library for:
- **GeoIP/GeoLite Web Services**: Country, City Plus, and Insights endpoints
- **GeoIP/GeoLite Databases**: Local MMDB file reading for various database types (City, Country, ASN, Anonymous IP, Anonymous Plus, ISP, etc.)

The library provides both web service clients and database readers that return strongly-typed model objects containing geographic, ISP, anonymizer, and other IP-related data.

**Key Technologies:**
- Ruby 3.2+ (uses frozen string literals and modern Ruby features)
- MaxMind DB Reader for binary database files
- HTTP gem for web service client functionality
- Minitest for testing
- RuboCop with multiple plugins for code quality

## Code Architecture

### Package Structure

```
lib/maxmind/geoip2/
├── model/              # Response models (City, Insights, AnonymousIP, etc.)
├── record/             # Data records (City, Location, Traits, etc.)
├── client.rb           # HTTP client for MaxMind web services
├── reader.rb           # Local MMDB file reader
├── errors.rb           # Custom exceptions for error handling
└── version.rb          # Version constant
```

### Key Design Patterns

#### 1. **Attr Reader Pattern for Immutable Data**

Models expose data through `attr_reader` attributes that are initialized in the constructor. Unlike PHP's readonly properties, Ruby uses instance variables with attr_reader:

```ruby
class City < Country
  attr_reader :city
  attr_reader :location
  attr_reader :postal
  attr_reader :subdivisions

  def initialize(record, locales)
    super
    @city = MaxMind::GeoIP2::Record::City.new(record['city'], locales)
    @location = MaxMind::GeoIP2::Record::Location.new(record['location'])
    @postal = MaxMind::GeoIP2::Record::Postal.new(record['postal'])
    @subdivisions = create_subdivisions(record['subdivisions'], locales)
  end
end
```

**Key Points:**
- Instance variables are set in the constructor
- Use `attr_reader` to expose them
- Models and records are initialized from hash data (from JSON/DB)
- Records are composed objects (City contains City record, Location record, etc.)

#### 2. **Inheritance Hierarchies**

Models follow clear inheritance patterns:
- `Country` → base model with country/continent data
- `City` extends `Country` → adds city, location, postal, subdivisions
- `Insights` extends `City` → adds additional web service fields (web service only)
- `Enterprise` extends `City` → adds enterprise-specific fields

Records have similar patterns:
- `Abstract` → base with `get` method for accessing hash data
- `Place` extends `Abstract` → adds names/locales handling
- Specific records (`City`, `Country`, etc.) extend `Place` or `Abstract`

#### 3. **Get Method Pattern for Data Access**

Both models and records use a protected `get` method to safely access hash data:

```ruby
def get(key)
  if @record.nil? || !@record.key?(key)
    return false if key.start_with?('is_')
    return nil
  end

  @record[key]
end
```

- Returns `false` for missing boolean fields (starting with `is_`)
- Returns `nil` for missing optional fields
- Records store the raw hash in `@record` instance variable

Public methods expose data through the `get` method:

```ruby
def anonymizer_confidence
  get('anonymizer_confidence')
end

def provider_name
  get('provider_name')
end
```

#### 4. **Lazy Parsing for Special Types**

Some fields require parsing and are computed lazily:

```ruby
def network_last_seen
  return @network_last_seen if defined?(@network_last_seen)

  date_string = get('network_last_seen')

  if !date_string
    @network_last_seen = nil
    return nil
  end

  @network_last_seen = Date.parse(date_string)
end
```

- Use `defined?(@variable)` to check if already parsed
- Parse only once and cache in instance variable
- Handle nil cases before parsing
- Memoize nil results too, so repeated calls do not re-run the lookup

#### 5. **Web Service Only vs Database Models**

Some models are only used by web services and do **not** need MaxMind DB support:

**Web Service Only Models**:
- Models that are exclusive to web service responses
- Simpler implementation, just inherit and define in model hierarchy
- Example: `Insights` (extends City but used only for web service)

**Database-Supported Models**:
- Models used by both web services and database files
- Reader has specific methods (e.g., `anonymous_ip`, `anonymous_plus`, `city`)
- Must handle MaxMind DB format data structures
- Example: `City`, `Country`, `AnonymousIP`, `AnonymousPlus`

## Testing Conventions

### Running Tests

```bash
# Install dependencies
bundle install

# Run all tests
bundle exec rake test

# Run tests and RuboCop
bundle exec rake  # default task

# Run RuboCop only
bundle exec rake rubocop

# Run specific test file
ruby -Ilib:test test/test_reader.rb
```

### Test Structure

Tests are organized by functionality:
- `test/test_reader.rb` - Database reader tests
- `test/test_client.rb` - Web service client tests
- `test/test_model_*.rb` - Model-specific tests
- `test/data/` - Test fixtures and sample database files

### Test Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxmind/GeoIP2-ruby](https://github.com/maxmind/GeoIP2-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
