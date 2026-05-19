---
trigger: always_on
description: Interactive pattern for adding feature flags to iOS and/or macOS with proper configuration
---


# Feature Flag Addition Pattern

## When This Pattern Applies

This pattern is activated when the user explicitly requests to add a feature flag, such as:
- "Add a feature flag for [feature name]"
- "Create a feature flag for [feature name] on [platform]"
- "I need a feature flag to control [feature name]"

## Overview

Adding a feature flag requires careful consideration of several factors:
1. **Platform** (iOS, macOS, or both)
2. **Source type** (how the flag is controlled)
3. **Default value** (fallback behavior)
4. **Local overriding** (debug menu access)
5. **Remote configuration** (if applicable)

## Step 1: Validate and Check for Duplicates

Before adding a new feature flag, check if a similar flag already exists:

```bash
# Search for similar flags
grep -i "case.*[searchTerm]" iOS/Core/FeatureFlag.swift
grep -i "case.*[searchTerm]" macOS/LocalPackages/FeatureFlags/Sources/FeatureFlags/FeatureFlag.swift
```

## Step 1.5: Create Asana Task (REQUIRED)

**STOP:** Before proceeding with implementation, the user must create an Asana task.

Instruct the user:
```
Please create an Asana task in the Apple Feature Flags Registry:

1. Open Asana
2. Navigate to the "Apple Feature Flags Registry" project
3. Create a new task default feature flag task
4. Copy the task URL

Paste the Asana task URL when ready to continue.
```

**This is mandatory** - all feature flags must be tracked in the Apple Feature Flags Registry.

## Step 2: Ask Clarifying Questions

### Question 1: Platform Selection

**Ask the user:**
```
Which platform(s) should this feature flag target?
  a) iOS only
  b) macOS only
  c) Both iOS and macOS
```

**Default:** Infer from user's request. If ambiguous, ask.

### Question 2: Feature Flag Source Type

**Ask the user:**
```
What source type should this feature flag use?

  a) .remoteReleasable - Can be controlled remotely in production (RECOMMENDED for most features)
     • Allows gradual rollout
     • Can be toggled without app updates
     • Requires Privacy Config setup
     
  b) .remoteDevelopment - Remote control in development environments only
     • For testing remote config before production
     • Not visible in production builds
     
  c) .internalOnly() - Only enabled for internal users
     • Always on for internal users
     • Always off for external users
     • No remote control
     
  d) .disabled - Always off for everyone
     • Placeholder for future features
     • Code is present but inactive

Which option? (a is recommended for new features)
```

**Important:** If user selects `a` or `b`, proceed to Question 2b.

### Question 2b: Parent Feature Selection (for remote flags)

**Ask the user:**
```
For remote feature flags, we need to add a subfeature to PrivacyFeature.swift.

Which parent feature should this belong to?

Platform-specific generic:
  a) macOSBrowserConfig - Generic macOS browser features
  b) iOSBrowserConfig - Generic iOS browser features

Domain-specific (if applicable):
  c) aiChat - AI Chat related features
  d) sync - Sync related features
  e) privacyPro - Privacy Pro subscription features
  f) autofill - Autofill related features
  g) networkProtection - VPN related features
  h) duckPlayer - Duck Player features
  i) dbp - Data Broker Protection features
  j) htmlNewTabPage - New Tab Page features
  k) maliciousSiteProtection - Malicious site protection
  l) Other existing parent feature (specify name)
  m) Create NEW parent feature (requires additional setup)

Which option?
```

**Guidance for selection:**
- Use platform-specific generic (a/b) when feature doesn't fit existing domains
- Use domain-specific when feature clearly belongs to an existing area
- Creating a new parent feature (m) requires:
  1. Adding case to `PrivacyFeature` enum
  2. Creating new `[FeatureName]Subfeature` enum
  3. Coordinating with backend team for remote config

### Question 3: Default Value

**Ask the user:**
```
What should the default value be?

  a) false - Feature OFF when remote config unavailable (RECOMMENDED)
     • Safer option
     • Opt-in behavior
     • Better for new/experimental features
     
  b) true - Feature ON when remote config unavailable
     • Used when feature should be on by default
     • Useful for rollback safety (can disable remotely)
     • Better for stable features being gradually enabled

Which option? (a is recommended for new features)
```

**Explanation:** The default value is used when:
- Remote config is unavailable
- Flag source is local-only (`.internalOnly`, `.disabled`)
- Network is down or config fetch fails

### Question 4: Local Overriding

**Ask the user:**
```
Should this feature flag support local overriding?

  a) true - Allow internal users to toggle in debug menu (RECOMMENDED)
     • Enables testing both states
     • Useful during development
     • No effect on external users
     
  b) false - No local override available
     • Use for production pixels/metrics
     • Use for security-critical flags
     • Use when override would break functionality

Which option? (a is recommended unless there's a specific reason)
```

### Question 5: Asana Task Link

**REQUIRED:** Before proceeding, the user must create an Asana task.

**Instruct the user:**
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
