---
trigger: always_on
description: The Conversions API Parameter Builder SDK is a multi-language toolkit designed
---

# AI Agent Guide for Conversions API Parameter Builder SDK

## Project Overview

The Conversions API Parameter Builder SDK is a multi-language toolkit designed
to improve the quality of Conversions API parameter retrieval and validation.
The SDK ensures event parameters adhere to best practices across multiple
platforms.

**Repository:** https://github.com/facebook/capi-param-builder

### Supported Languages

- **Server-side:** PHP, Java, NodeJS, Python, Ruby
- **Client-side:** JavaScript

Each language implementation lives in its own top-level directory with its own
build system, tests, examples, and documentation.

## Project Structure

```
param_sdk/
├── README.md              # Main project documentation
├── AGENTS.md              # AI Agent guidance (this file)
├── CODE_OF_CONDUCT.md     # Code of conduct
├── CONTRIBUTING.md        # Contribution guidelines
├── LICENSE                # License file
├── composer.json          # PHP root composer file
├── .github/workflows/     # CI/CD workflows for all languages
├── java/                  # Java implementation
├── nodejs/                # Node.js implementation
├── php/                   # PHP implementation
├── python/                # Python implementation
└── ruby/                  # Ruby implementation
```

**Note:** The `client_js/` directory is **not open-sourced** yet, the dir only
contains examples.

Each language directory follows a consistent structure:

```
<language>/
├── README.md              # Language-specific quick start guide
├── CHANGELOG.md           # Version history and changes
├── CONTRIBUTING.md        # Contribution guidelines
├── CODE_OF_CONDUCT.md     # Code of conduct
├── LICENSE                # License file
├── capi-param-builder/    # Main SDK implementation (Python/Ruby use capi_param_builder with underscore)
│   ├── src/               # Source code
│   ├── tests/ or test/    # Unit tests
│   └── ...                # Language-specific build files
└── example/ or examples/  # Example usage code (PHP uses "examples")
```

## Core Functionality

The SDK provides utilities for:

- **Cookie parameter extraction** - Facebook click identifier (fbc) and browser
  identifier (fbp)
- **Client IP address retrieval** - Enhanced mechanisms for capturing client IP
  addresses with IPv6 support (IPv4 fallback when IPv6 is unavailable)
- **Customer Information Parameters normalization and hashing** - Tools to help
  adopt best practices for normalizing and hashing customer information
  including email, phone, name (first and last), address (city, state, zip code,
  country), gender, date of birth, external ID

For details on features, refer to the
[Parameter Builder Library](https://developers.facebook.com/docs/marketing-api/conversions-api/parameter-builder-feature-library).

## Testing and Validation

### Critical Rule: Always Run Tests Before Committing Changes

**MANDATORY:** After making any code changes to a language implementation, you
MUST run the appropriate test command for that language before considering the
task complete.

### Test Commands by Language

#### Node.js

```bash
cd nodejs/capi-param-builder
npm install
npm test
```

Tested on Node.js versions: 18.x, 20.x, 22.x, 24.x

#### Python

```bash
cd python/capi_param_builder
python3 -m unittest test/test_param_builder.py
```

Tested on Python versions: 3.9, 3.10, 3.11

#### Java

```bash
cd java/capi-param-builder
chmod +x ./gradlew
./gradlew build
```

Tested on Java versions: 8, 11, 17, 21

#### PHP

```bash
cd php/capi-param-builder
composer install --dev --prefer-source
./vendor/bin/phpunit ./tests/ --debug
```

Tested on PHP versions: 7.4, 8.0, 8.1, 8.2, 8.3, 8.4

#### Ruby

```bash
cd ruby/capi_param_builder
gem install minitest
ruby -Ilib:test test/test_param_builder.rb
```

Tested on Ruby versions: 3.0, 3.2, 3.3

## Making Code Changes

### Language-Specific Considerations

1. **Maintain API Consistency**: All language implementations should provide
   equivalent functionality with language-appropriate idioms. When adding a
   feature to one language, consider whether it should be added to others.

2. **Backward Compatibility**: The SDK is used by external developers. We
   strongly recommend maintaining backward compatibility with existing flows.
   Breaking changes require:
   - Version bump (follow semantic versioning)
   - Changelog entry
   - Migration guide if necessary

3. **Human Review for Intrusive Changes**: If you have intrusive changes that
   could break existing integrations or significantly alter SDK behavior, please
   ensure human review before implementation. Always prefer backward-compatible
   solutions over breaking changes.

### Common File Locations

**Note:** Python and Ruby use `capi_param_builder` (with underscores) instead of
`capi-param-builder` (with dashes).

- **Core logic:** `<language>/capi-param-builder/src/` (or `capi_param_builder/`
  for Python/Ruby)
- **Tests:** `<language>/capi-param-builder/tests/` or
  `<language>/capi-param-builder/test/` (or `capi_param_builder/` for
  Python/Ruby)
- **Models:** `<language>/capi-param-builder/src/model/` or similar (or
  `capi_param_builder/` for Python/Ruby)
- **Utilities:** `<language>/capi-param-builder/src/utils/` (or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/capi-param-builder](https://github.com/facebook/capi-param-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
