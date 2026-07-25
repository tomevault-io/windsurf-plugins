---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PhoneNumberKit is a Swift framework for parsing, formatting, and validating international phone numbers. It's inspired by Google's libphonenumber and supports iOS, macOS, tvOS, and watchOS platforms.

## Common Commands

### Testing
```bash
# Run tests via Swift Package Manager (works on Linux and macOS)
swift test

# Run tests for iOS using Xcode
xcodebuild test -scheme PhoneNumberKit -destination 'platform=iOS Simulator,name=iPhone 15'

# Run tests for macOS using Xcode
xcodebuild test -scheme PhoneNumberKit-macOS -destination 'platform=macOS'
```

### Building
```bash
# Build via Swift Package Manager
swift build

# Build for iOS (Carthage-style)
xcodebuild -project PhoneNumberKit.xcodeproj -scheme PhoneNumberKit -destination 'generic/platform=iOS' build

# Build for macOS
xcodebuild -project PhoneNumberKit.xcodeproj -scheme PhoneNumberKit-macOS -destination 'generic/platform=macOS' build

# Build for tvOS
xcodebuild -project PhoneNumberKit.xcodeproj -scheme PhoneNumberKit-tvOS -destination 'generic/platform=tvOS' build

# Build for watchOS
xcodebuild -project PhoneNumberKit.xcodeproj -scheme PhoneNumberKit-watchOS -destination 'generic/platform=watchOS' build
```

### CocoaPods
```bash
# Validate podspec
pod lib lint PhoneNumberKit.podspec

# Push to CocoaPods trunk
pod trunk push PhoneNumberKit.podspec
```

### Metadata Updates
```bash
# Update phone number metadata from Google's libphonenumber (requires Python 3.12+ and xmljson)
cd PhoneNumberKit/Resources && sh ./update_metadata.sh
```

## Architecture

### Core Components

**PhoneNumberUtility** (PhoneNumberKit/PhoneNumberUtility.swift)
- The main entry point for all phone number operations
- Relatively expensive to allocate (parses and keeps metadata in memory)
- Should be instantiated once and reused throughout the app lifecycle
- Provides parsing, formatting, validation, and country/region code queries

**PhoneNumber** (PhoneNumberKit/PhoneNumber.swift)
- Immutable struct representing a parsed phone number
- Contains: numberString, countryCode, leadingZero, nationalNumber, numberExtension, type, regionID
- Implements Equatable, Hashable, and Codable

**MetadataManager** (PhoneNumberKit/MetadataManager.swift)
- Loads and manages phone number metadata from PhoneNumberMetadata.json
- Maintains dictionaries for fast lookup by country code and country ID
- Handles territories with multiple countries sharing calling codes (e.g., NANPA)

**PhoneNumberParser** (PhoneNumberKit/PhoneNumberParser.swift)
- Internal parser handling normalization, extraction, and validation
- Strips international/national prefixes and extensions
- Determines phone number type (mobile, fixed line, etc.)

**ParseManager** (PhoneNumberKit/ParseManager.swift)
- Coordinates between parser and metadata manager
- Handles single and batch parsing operations

### Formatting Components

**Formatter** (PhoneNumberKit/Formatter.swift)
- Formats PhoneNumber objects to different formats: E164, international, national

**PartialFormatter** (PhoneNumberKit/PartialFormatter.swift)
- Provides as-you-type formatting for user input
- Used internally by PhoneNumberTextField
- Only available on platforms with ObjectiveC support (not available on Linux)

### UI Components (iOS only)

**PhoneNumberTextField** (PhoneNumberKit/UI/PhoneNumberTextField.swift)
- Custom UITextField with automatic phone number formatting
- Features: flag display, example placeholder, prefix handling
- Integrates PartialFormatter for real-time formatting

**CountryCodePickerViewController** (PhoneNumberKit/UI/CountryCodePickerViewController.swift)
- Country code picker UI for PhoneNumberTextField
- Customizable via CountryCodePickerOptions

## Key Design Patterns

### Metadata Loading
- Metadata is loaded once during PhoneNumberUtility initialization via a callback
- Default callback reads from bundled PhoneNumberMetadata.json
- Custom metadata sources can be provided via metadataCallback parameter

### Region Detection
- Uses CNContactsUserDefaults.shared().countryCode for iOS 12+
- Falls back to Locale.current for region detection
- Handles edge cases (e.g., Korean language on macCatalyst returns "ko" instead of "kr")

### Type Validation
- Parser checks number against metadata patterns for each phone type
- Can be skipped via `ignoreType` parameter for performance
- Type hierarchy: specific types (mobile, pager, etc.) → fixedOrMobile → unknown

### Performance Optimization
- Batch parsing with `parse(_:withRegion:ignoreType:)` for arrays
- Regex patterns are cached in RegexManager
- Metadata dictionaries provide O(1) lookup by code/country

## Platform Specifics

### Swift Package Manager
- Target: `PhoneNumberKit` at PhoneNumberKit/
- Resources: PhoneNumberMetadata.json, PrivacyInfo.xcprivacy
- Test target: `PhoneNumberKitTests` at PhoneNumberKitTests/
- Supports iOS 12+, macOS 10.13+, tvOS 12+, watchOS 4+

### CocoaPods
- Two subspecs: PhoneNumberKitCore (core parsing) and UIKit (UI components)
- UIKit subspec requires iOS 12+ and depends on PhoneNumberKitCore

### Xcode Project

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmelroy/PhoneNumberKit](https://github.com/marmelroy/PhoneNumberKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
