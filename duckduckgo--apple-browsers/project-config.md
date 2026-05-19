---
trigger: always_on
description: The DuckDuckGo browser includes a comprehensive A/B/N experiment framework that enables data-driven feature testing across iOS and macOS platforms. This framework allows you to safely experiment with new ideas while maintaining control groups and measuring impact.
---


# A/B/N Experiment Framework

## Overview

The DuckDuckGo browser includes a comprehensive A/B/N experiment framework that enables data-driven feature testing across iOS and macOS platforms. This framework allows you to safely experiment with new ideas while maintaining control groups and measuring impact.

**Reference**: Video of knowledge sharing session: ✓ A/B/N Experiment Framework

## What is A/B/N Testing?

An **A/B/N test** is a method of experimenting with multiple variants of a feature (A, B, ... N) to determine which performs better. It's like traditional A/B testing but scaled up to support more than two groups.

### Use Cases

You can use this framework to explore:
- **UI/UX variations**: Whether blue buttons outperform green buttons
- **Content experiments**: If showing pictures of cats 🐱 or dogs 🐶 boosts user retention
- **Feature comparisons**: Different implementations of the same functionality
- **Performance optimization**: Testing various algorithms or approaches

### When to Use A/B/N Testing

✅ **Use for**:
- Comparing user behavior between two or more feature variants
- Validating hypotheses before rolling out changes to all users
- Safely experimenting with new ideas while maintaining control groups
- Measurable, impactful decisions with clear success metrics

❌ **Don't use for**:
- Simple bug fixes or obvious improvements
- Changes without measurable impact
- Features that can't be easily reversed

⚠️ **Note**: Not every change needs a test—reserve it for measurable, impactful decisions. This is typically decided in collaboration with ODRIs and Data Science.

## Framework Architecture

### Remote Configuration System

A/B/N experiments are supported via **remote configuration** on both macOS and iOS:

- **Sub-features** are used for experiments
- **Parent features** group related experiments
- **Cohorts** define the different variants
- **Weights** control user distribution
- **Targets** allow locale-based segmentation

## Configuration Setup

### 1. Privacy Config Structure

Experiments are defined in the Privacy Configuration with this structure:

```json
"amazingMacroFeature": {
  "state": "enabled",
  "features": {
    "petsPictures": {
      "state": "enabled",
      "description": "This feature shows users pictures of cute pets",
      "targets": [
        { "localeLanguage": "en", "localeCountry": "US" },
        { "localeLanguage": "fr", "localeCountry": "CA" }
      ],
      "cohorts": [
        { "name": "cats", "weight": 1 },
        { "name": "dogs", "weight": 1 }
      ]
    }
  }
}
```

### Configuration Elements

#### **State Options**
- `enabled`: Visible to all users
- `internal`: Visible only to internal users  
- `disabled`: Hidden from all users

#### **Description**
Explains the experiment's purpose for team reference.

#### **Targets** (Optional)
Specify user segments based on locale:
```json
"targets": [
  { "localeLanguage": "en", "localeCountry": "US" },
  { "localeLanguage": "fr", "localeCountry": "CA" }
]
```

#### **Cohorts**
Define experiment variants:
- `name`: Cohort identifier (e.g., "cats", "dogs")
- `weight`: Probability of assignment (normally 1 or 0)

## Client Implementation

### Step 1: Add Feature to PrivacyFeature (BSK)

#### Check for Existing Features

```swift
// In PrivacyFeature enum, check if parent feature exists
public enum PrivacyFeature: String, CaseIterable {
    case amazingMacroFeature
    // ... other features
}

// Add sub-feature to existing enum or create new one
public enum AmazingMacroFeatureSubfeatures: String, CaseIterable {
    case petsPictures
    // ... other sub-features
}
```

#### Add New Features

If the parent feature doesn't exist:
1. Add it to the `PrivacyFeature` enum
2. Create a new sub-features enum
3. Add your sub-feature to the enum

### Step 2: Define Feature Flag

Add your experiment to the local `FeatureFlag` enum:

```swift
public enum FeatureFlag: String, CaseIterable {
    case debugMenu
    case sslCertificatesBypass
    case maliciousSiteProtection
    // ... existing flags
    case petsPictures

    public var cohortType: (any FeatureFlagCohortDescribing.Type)? {
        switch self {
        case .petsPictures:
            return PetsPicturesCohort.self
        default:
            return nil
        }
    }

    public enum PetsPicturesCohort: String, FeatureFlagCohortDescribing {
        case cats
        case dogs
    }

    public var source: FeatureFlagSource {
        switch self {
        // ... other cases
        case .petsPictures:
            return .remoteReleasable(.subfeature(AmazingMacroFeatureSubfeatures.petsPictures))
        }
    }

    public var supportsLocalOverriding: Bool {
        switch self {
        // ... other cases
        case .petsPictures: 
            return true
        }
    }
}
```

#### Key Properties

**`cohortType`**: Links to experiment cohorts enum
- Must conform to `String, FeatureFlagCohortDescribing`
- Defines available variants (cats, dogs)

**`source`**: Defines feature flag toggle location
- `.disabled`: Feature is off

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
