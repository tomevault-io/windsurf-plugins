---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Celestra is a command-line RSS reader that demonstrates MistKit's CloudKit integration capabilities. It fetches RSS feeds, stores them in CloudKit's public database, and implements comprehensive web etiquette best practices including rate limiting, robots.txt checking, and conditional HTTP requests.

**Tech Stack**: Swift 6.2, MistKit (CloudKit wrapper), CelestraKit (shared models & services), SyndiKit (RSS parsing), Swift Configuration (configuration management)

## Common Commands

### Build and Run

```bash
# Build the project
swift build

# Run with environment variables
source .env
swift run celestra-cloud <command>

# Add a feed
swift run celestra-cloud add-feed https://example.com/feed.xml

# Update feeds with filters
swift run celestra-cloud update
swift run celestra-cloud update --update-last-attempted-before 2025-01-01T00:00:00Z
swift run celestra-cloud update --update-min-popularity 10 --update-delay 3.0
swift run celestra-cloud update --update-limit 5 --update-max-failures 0

# Clear all data
swift run celestra-cloud clear --confirm

# Using both environment variables and CLI arguments (CLI wins)
UPDATE_DELAY=2.0 swift run celestra-cloud update --update-delay 3.0
```

### Environment Setup

Required environment variables (see `.env.example`):
- `CLOUDKIT_KEY_ID` - Server-to-Server key ID from Apple Developer Console
- `CLOUDKIT_PRIVATE_KEY_PATH` - Path to `.pem` private key file

Optional environment variables:
- `CLOUDKIT_CONTAINER_ID` - CloudKit container identifier (default: `iCloud.com.brightdigit.Celestra`)
- `CLOUDKIT_ENVIRONMENT` - Either `development` or `production` (default: `development`)

### CloudKit Schema Management

```bash
# Automated schema deployment (requires cktool)
export CLOUDKIT_CONTAINER_ID="iCloud.com.brightdigit.Celestra"
export CLOUDKIT_TEAM_ID="YOUR_TEAM_ID"
export CLOUDKIT_ENVIRONMENT="development"
./Scripts/setup-cloudkit-schema.sh
```

Schema is defined in `schema.ckdb` using CloudKit's text-based schema language.

## Architecture

### High-Level Structure

```
Sources/CelestraCloud/
├── Celestra.swift              # CLI entry point
└── Commands/                   # CLI subcommands
    ├── AddFeedCommand.swift    # Parse and add RSS feeds
    ├── UpdateCommand.swift     # Fetch/update feeds (shows MistKit QueryFilter)
    └── ClearCommand.swift      # Delete all records

Sources/CelestraCloudKit/
├── Configuration/              # Swift Configuration integration
│   ├── CelestraConfiguration.swift          # Root config struct
│   ├── CloudKitConfiguration.swift          # CloudKit credentials config
│   ├── UpdateCommandConfiguration.swift     # Update command options
│   ├── ConfigurationLoader.swift            # Multi-source config loader
│   └── ConfigurationError.swift             # Enhanced errors
├── CelestraConfig.swift        # CloudKit service factory
├── Services/
│   ├── CloudKitService+Celestra.swift  # MistKit operations
│   ├── CelestraError.swift             # Error types
│   └── CelestraLogger.swift            # Structured logging
├── Models/
│   └── BatchOperationResult.swift      # Batch operation tracking
└── Extensions/
    ├── Feed+MistKit.swift      # Feed ↔ CloudKit conversion
    └── Article+MistKit.swift   # Article ↔ CloudKit conversion
```

**External Dependencies**: The `Feed` and `Article` models, along with `RateLimiter` and `RobotsTxtService`, are provided by the CelestraKit package for reuse across CLI and other clients.

### Key Architectural Patterns

**1. MistKit Integration**

CloudKitService is configured in `CelestraConfig.createCloudKitService()`:
- Server-to-Server authentication using PEM keys
- Public database access for shared feeds
- Environment-based configuration (dev/prod)

All CloudKit operations are in `CloudKitService+Celestra.swift` extension:
- `queryFeeds()` - Demonstrates QueryFilter and QuerySort APIs
- `createArticles()` / `updateArticles()` - Batch operations with chunking
- `queryArticlesByGUIDs()` - Duplicate detection queries

**2. Field Mapping Pattern**

Models use direct field mapping with validation (CloudKitConvertible protocol):

```swift
// To CloudKit
func toFieldsDict() -> [String: FieldValue] {
    var fields: [String: FieldValue] = [
        "title": .string(title),
        "isActive": .int64(isActive ? 1 : 0)  // Booleans as INT64
    ]
    // Optional fields only added if present
    if let description = description {
        fields["description"] = .string(description)
    }
    return fields
}

// From CloudKit - with validation (throws CloudKitConversionError)
init(from record: RecordInfo) throws {
    // Required fields throw if missing or empty
    guard case .string(let title) = record.fields["title"],
          !title.isEmpty else {
        throw CloudKitConversionError.missingRequiredField(
            fieldName: "title",
            recordType: "Feed"
        )
    }

    // Boolean extraction with default
    if case .int64(let value) = record.fields["isActive"] {
        self.isActive = value != 0
    } else {
        self.isActive = true  // Default for optional fields
    }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brightdigit/MistKit](https://github.com/brightdigit/MistKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
