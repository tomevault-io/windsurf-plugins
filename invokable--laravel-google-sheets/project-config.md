---
trigger: always_on
description: Transforms raw data into Laravel Collection with header-based associative arrays.
---

# Onboarding Guide: Laravel Google Sheets Integration Package

## Overview

**revolution/laravel-google-sheets** is a Laravel package that provides a simplified, Laravel-idiomatic interface for interacting with Google Sheets API v4. The package abstracts the complexities of the underlying Google API PHP client library, offering developers a streamlined way to perform common spreadsheet operations within Laravel applications.

**Target Users**: Laravel developers who need to integrate Google Sheets functionality into their web applications.

**Key Capabilities**:
- **Data Operations**: Read, write, update, and clear spreadsheet data with fluent API methods
- **Sheet Management**: Create, delete, and list sheets within spreadsheets
- **Authentication Flexibility**: Supports OAuth 2.0 (user-specific), Service Accounts (server-to-server), and API keys (public data)
- **Laravel Integration**: Provides facades, traits for Eloquent models, and service providers for seamless framework integration
- **Data Transformation**: Converts raw Google API responses into Laravel Collections for easy manipulation
- **Extensibility**: Macro system allows custom method addition to the main facade

**Primary Use Cases**:
- User dashboards that display Google Sheets data
- Data import/export functionality
- Automated reporting systems
- Multi-user applications where each user manages their own spreadsheets

## Copilot Environment Restrictions

⚠️ **CRITICAL**: The Copilot firewall causes errors when connecting to `www.googleapis.com`

- Google API calls will fail in the Copilot development environment
- This affects testing and development of Google Sheets functionality
- GitHub Actions and production environments have no such restrictions
- **Workaround**: Use mock data or skip API calls when testing in Copilot environment
- Real API functionality should be tested in GitHub Actions or local development environments

**Note**: All examples and tests should be designed to handle API connectivity failures gracefully when running in restricted environments.

## Project Organization

### Core Systems Architecture

The package is organized around several interconnected systems:

1. **Core Interface Layer** (`src/`)
    - `SheetsClient.php` - Main implementation class that handles Google Sheets operations
    - `Facades/Sheets.php` - Laravel facade providing static access to SheetsClient
    - `Contracts/Factory.php` - Interface defining the core contract for Google Sheets operations

2. **Modular Functionality** (`src/Concerns/`)
    - `SheetsValues.php` - CRUD operations (create, read, update, delete) for spreadsheet data
    - `SheetsCollection.php` - Data transformation utilities for converting API responses to Laravel Collections
    - `SheetsProperties.php` - Methods for retrieving spreadsheet and sheet metadata
    - `SheetsDrive.php` - Google Drive integration for spreadsheet management

3. **Google API Integration** (`lib/google/`)
    - `GoogleApiClient.php` - Wrapper around Google's PHP client library
    - `Facades/Google.php` - Facade for Google API client access
    - `Providers/GoogleServiceProvider.php` - Service provider for Google client registration

4. **Laravel Integration** (`src/`)
    - `Providers/SheetsServiceProvider.php` - Main service provider for package registration
    - `Traits/GoogleSheets.php` - Trait for Eloquent models to enable user-specific Google Sheets access

5. **Configuration & Documentation**
    - `config/google.php` - Configuration template for Google API credentials
    - `docs/` - Usage documentation and examples
    - `composer.json` - Package definition with dependencies and autoloading

### Main Directories

```
├── src/                          # Core package source code
│   ├── Concerns/                 # Trait-based modular functionality
│   ├── Contracts/               # Interface definitions
│   ├── Facades/                 # Laravel facades
│   ├── Providers/               # Service providers
│   └── Traits/                  # Traits for model integration
├── lib/google/                  # Google API client wrapper
│   ├── Facades/
│   └── Providers/
├── tests/                       # Comprehensive test suite
├── docs/                        # Documentation and usage examples
├── .github/workflows/           # CI/CD automation
├── config/                      # Configuration templates
└── composer.json               # Package definition
```

### Development Practices

**Testing Strategy**:
- **Unit Tests**: Individual component testing (e.g., `SheetsCollectionTest.php`)
- **Integration Tests**: Facade and trait functionality testing (e.g., `SheetsTest.php`)
- **Mock Tests**: Google API interaction testing with mocked responses (e.g., `SheetsMockTest.php`)
- **Test Infrastructure**: Uses Orchestra Testbench for Laravel package testing and Mockery for API mocking

**CI/CD Pipeline** (`.github/workflows/`):
- **test.yml**: Runs PHPUnit tests across multiple PHP versions (8.2, 8.3, 8.4) with code coverage reporting
- **lint.yml**: Enforces code style using Laravel Pint on develop and main branches

**Code Quality**:
- PHP 8.2+ type declarations and features
- PSR-4 autoloading with clear namespace organization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [invokable/laravel-google-sheets](https://github.com/invokable/laravel-google-sheets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
