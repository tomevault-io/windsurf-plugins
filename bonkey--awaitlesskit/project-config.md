---
trigger: always_on
description: AwaitlessKit is a Swift package providing macros to automatically generate synchronous wrappers for async/await functions, enabling easy migration to Swift 6 Structured Concurrency with both APIs available.
---

# AwaitlessKit

AwaitlessKit is a Swift package providing macros to automatically generate synchronous wrappers for async/await functions, enabling easy migration to Swift 6 Structured Concurrency with both APIs available.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites

- Swift 6.0+ compiler required
- This package works on Linux and macOS
- Do NOT install mise or just tools, but do use Justfile CONTENTS to identify commands to run, so if instructions say "just test", check the contents of Justfile for "test" command contents and run it without just.

### Bootstrap and Build

- `cd /path/to/AwaitlessKit` - always work from repository root
- `just package-build` - builds all targets including macros. Takes 3+ minutes initially. NEVER CANCEL. Set timeout to 10+ minutes.
- `just package-resolve` - resolves dependencies if needed (normally automatic)

### Testing

- `just package-test` - runs full test suite. Takes 35-60 seconds. NEVER CANCEL. Set timeout to 5+ minutes.
- `just package-test TestSuiteName` - runs specific test suite
- All tests should pass on Linux and macOS

### Documentation

- The `README.md` file must always be up to date with the code. However, it should not cover all cases to prevent it from becoming too large.
- **ALWAYS update version numbers in README.md** when making releases or version-related changes
- Do always update DocC packages (e.g. `Sources/AwaitlessKit/Documentation.docc` and )
- Do always create and update articles in DocC packages (e.g. `Sources/AwaitlessKitMacros/AwaitlessKitMacros.docc` and `Sources/AwaitlessKit/AwaitlessKit.docc`). They must be very comprehensive and include high-level information as well.
- When updating documentation links, always use 'main' branch URLs instead of version-specific URLs
- **Remember**: All new features require comprehensive documentation updates (see [New Feature Requirements](#new-feature-requirements))

### Versioning

- Use semantic versioning (major.minor.patch) for releases.
  - Increment major version for breaking changes.
  - Increment minor version for new features.
  - Increment patch version for bug fixes.
- Always suggest proper version in each PR, depending on the scale of changes
- **CRITICAL**: When updating versions, update ALL of the following locations:
  - `README.md` - Package.swift dependency version (documentation URLs should use 'main' branch)
  - Any documentation links should reference 'main' branch, not specific version numbers
  - Example code in DocC comments that show version numbers

### Code Validation

- `just fmt` - formats code using SwiftFormat
- CI runs on Linux and macOS with GitHub Actions

### New Feature Requirements

When adding any new features to AwaitlessKit, **ALWAYS** ensure both of the following requirements are met:

1. **SampleApp Integration**: Add the new feature to `SampleApp/` with working examples
   - Create realistic usage examples that demonstrate the feature
   - Ensure examples compile and run correctly on macOS
   - Examples should be comprehensive enough to serve as documentation
   - Use the feature in a real-world context within the sample application

2. **Documentation Updates**: Update all relevant documentation
   - Add feature description and usage examples to `README.md`
   - Update DocC documentation packages with comprehensive guides
   - Include code examples in documentation comments
   - Update version references if this is a major/minor release
   - Ensure documentation covers edge cases and limitations

**Note**: These requirements apply to any new macros, APIs, configuration options, or functionality changes. Even small features should be demonstrated in SampleApp and documented in README.

## Validation Scenarios

### Core Macro Functionality Testing

After making changes to macro code, always validate by:

1. `just build` - ensure macros compile without errors
2. `just test` - run full test suite (tests are fast, no need to filter)

### Integration Testing

- SampleApp is an Xcode project located in `SampleApp/` directory
- **SampleApp CANNOT be built or run on Linux** - requires Xcode and macOS
- Use test suite instead of SampleApp for validation on Linux systems
- SampleApp demonstrates real-world usage patterns for documentation
- **Remember**: All new features must be added to SampleApp (see [New Feature Requirements](#new-feature-requirements))

**Note**:

- The @Awaitless macro only works on class/struct methods, not global functions
- The macro generates synchronous versions with same names but different availability attributes
- Always use the comprehensive test suite for validation rather than manual test packages

## Repository Structure

### Key Directories

```
Sources/
├── AwaitlessKit/           # Main library with macro definitions
├── AwaitlessKitMacros/     # Macro implementation using SwiftSyntax
└── AwaitlessCore/          # Shared types and enums

Tests/
└── AwaitlessKitTests/      # Comprehensive test suite


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bonkey/AwaitlessKit](https://github.com/bonkey/AwaitlessKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
