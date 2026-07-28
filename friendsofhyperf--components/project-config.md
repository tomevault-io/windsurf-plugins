---
trigger: always_on
description: This repository contains a monorepo of popular components for the Hyperf PHP framework. This document provides guidance for GitHub Copilot to better understand the project structure and conventions.
---

# GitHub Copilot Instructions for FriendsOfHyperf Components

This repository contains a monorepo of popular components for the Hyperf PHP framework. This document provides guidance for GitHub Copilot to better understand the project structure and conventions.

> **Note**: Also see `CLAUDE.md` and `AGENTS.md` in the repository root for additional development guidance.

## Project Overview

- **Repository**: friendsofhyperf/components
- **Type**: Monorepo containing 49 Hyperf components
- **PHP Version**: >=8.2
- **Framework**: Hyperf >=3.2.0
- **Testing Framework**: Pest (PHPUnit under the hood)
- **License**: MIT
- **Main Author**: Deeka Wong (huangdijia@gmail.com)

## Repository Structure

```
/
├── bin/                    # Utility scripts
├── docs/                   # Multi-language documentation (VitePress)
├── src/                    # Component source code (49 components)
├── tests/                  # Test suites (Pest)
├── types/                  # PHP type stubs
├── .github/                # GitHub workflows and configuration
├── composer.json           # Main composer configuration
├── package.json            # Documentation build configuration
├── phpstan.neon.dist       # PHPStan configuration
├── CLAUDE.md               # Claude Code (claude.ai/code) development guidance
└── AGENTS.md               # Repository guidelines for agents
```

## Component Architecture

Each component in `src/` follows this structure:
```
src/{component-name}/
├── .gitattributes
├── .github/                # Component-specific GitHub config
├── LICENSE
├── README.md
├── composer.json           # Component-specific dependencies
└── src/                    # Component source code
    ├── ConfigProvider.php  # Hyperf configuration provider
    └── [component files]
```

### Key Components Include:

**Development Tools**:
- **telescope**: Application debugging and monitoring
- **tinker**: Interactive REPL
- **ide-helper**: IDE autocompletion helper
- **pretty-console**: Enhanced console output

**Database**:
- **model-factory**: Database model factories
- **model-observer**: Model event observers
- **model-scope**: Model query scopes
- **fast-paginate**: Optimized pagination
- **compoships**: Composite key relationships
- **mysql-grammar-addon**: MySQL grammar extensions

**Caching/Storage**:
- **cache**: Enhanced caching functionality
- **lock**: Distributed locking
- **redis-subscriber**: Redis pub/sub subscriber

**HTTP/API**:
- **http-client**: HTTP client utilities
- **oauth2-server**: OAuth2 server implementation
- **openai-client**: OpenAI API client

**Notifications**:
- **notification**: Multi-channel notifications
- **notification-mail**: Email notifications
- **notification-easysms**: SMS notifications via EasySms
- **mail**: Mail sending functionality

**Search**:
- **elasticsearch**: Elasticsearch integration
- **telescope-elasticsearch**: Telescope Elasticsearch storage

**Security**:
- **encryption**: Encryption utilities
- **purifier**: HTML purification
- **recaptcha**: Google reCAPTCHA integration
- **validated-dto**: Data Transfer Objects with validation

**Infrastructure**:
- **confd**: Configuration daemon
- **config-consul**: Consul configuration center
- **facade**: Laravel-style facades
- **ipc-broadcaster**: IPC broadcasting
- **sentry**: Sentry error tracking integration

## Development Conventions

### Namespace Convention
All components use the namespace pattern: `FriendsOfHyperf\{ComponentName}`

### Configuration Providers
Each component typically includes a `ConfigProvider.php` file that defines:
- Dependencies
- Commands
- Listeners
- Annotations

### Code Style
- Follows PSR-12 coding standards
- Uses PHP-CS-Fixer for code formatting (`.php-cs-fixer.php`)
- PHPStan for static analysis (`phpstan.neon.dist`)

### Testing
- Uses Pest testing framework (PHPUnit under the hood)
- Test configuration in `phpunit.xml.dist` and `tests/Pest.php`
- Test files located in `tests/` directory with groups per component
- Follows Hyperf testing patterns
- Type coverage analysis via Pest plugin

## Documentation Structure

Documentation is available in multiple languages:
- `docs/zh-cn/` - Simplified Chinese (primary)
- `docs/zh-tw/` - Traditional Chinese
- `docs/zh-hk/` - Hong Kong Chinese  
- `docs/en/` - English

### Documentation Generation
- Uses VitePress for documentation site
- Configuration in `docs/.vitepress/config.mts`
- Auto-generated component docs via `bin/generate-repository-doc.sh`
- Translation scripts available in `bin/`

## Build and Deployment

### Scripts Available:
- `bin/generate-repository-doc.sh` - Generate component documentation
- `bin/regenerate-readme.sh` - Regenerate README files
- `bin/doc-translate` - PHP-based translation script (uses OpenCC)
- `bin/doc-translate.js` - JavaScript-based translation
- `bin/doc-translate.github-model.js` - AI-powered translation using language models
- `bin/split-docs.sh` - Deploy documentation to separate repository
- `bin/split.sh`, `bin/split-linux.sh` - Split components to individual repos
- `bin/release.sh` - Release management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [friendsofhyperf/components](https://github.com/friendsofhyperf/components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
