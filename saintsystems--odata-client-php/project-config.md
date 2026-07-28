---
trigger: always_on
description: OData Client for PHP is a fluent library for calling OData REST services, inspired by the Laravel Query Builder. This is a PHP library package (not an application) distributed via Composer/Packagist.
---

# OData Client for PHP

OData Client for PHP is a fluent library for calling OData REST services, inspired by the Laravel Query Builder. This is a PHP library package (not an application) distributed via Composer/Packagist.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Setup
- PHP 7.4 or higher required
- Composer required for dependency management
- Internet access required for dependency installation and testing (tests use external OData service)

### Bootstrap and Build Process
```bash
# Install production dependencies first (usually works within 2-3 minutes)
composer install --no-dev --no-interaction

# Then install dev dependencies (NEVER CANCEL: can take 10-20 minutes due to network issues)
composer install --no-interaction
```
**CRITICAL TIMING**: 
- Production dependencies: 2-3 minutes (usually successful)
- Full install with dev dependencies: 10-20 minutes due to GitHub API rate limits and authentication issues
- Set timeout to 30+ minutes for full installation
- NEVER CANCEL the installation process - timeouts are expected and Composer will retry with source downloads

### Running Tests
```bash
# First ensure all dependencies are installed
composer install --no-interaction

# Run all tests (NEVER CANCEL: takes 2-5 minutes due to external API calls)
vendor/bin/phpunit

# If PHPUnit is not available due to dependency installation issues:
php -f tests/ODataClientTest.php
```
**CRITICAL TIMING**: Tests make real HTTP calls to external OData service (services.odata.org/V4/TripPinService). Set timeout to 10+ minutes. NEVER CANCEL test execution.

**DEPENDENCY NOTE**: If dev dependencies fail to install due to network timeouts, you can still validate basic functionality with production dependencies only.

### Code Quality and Validation
```bash
# Static analysis (if PHPStan is available)
vendor/bin/phpstan analyse src/

# Syntax check all PHP files
find src/ -name "*.php" -exec php -l {} \;

# Check autoloading
composer dump-autoload
```

## Validation Scenarios

After making changes to this library, ALWAYS test the following scenarios:

### Core Functionality Validation
```php
<?php
require_once 'vendor/autoload.php';

use SaintSystems\OData\ODataClient;
use SaintSystems\OData\GuzzleHttpProvider;

// Basic validation script (works with production dependencies only):
echo "Testing class loading...\n";
echo "✓ ODataClient: " . (class_exists('SaintSystems\OData\ODataClient') ? 'Found' : 'NOT FOUND') . "\n";
echo "✓ Entity: " . (class_exists('SaintSystems\OData\Entity') ? 'Found' : 'NOT FOUND') . "\n";
echo "✓ Query\\Builder: " . (class_exists('SaintSystems\OData\Query\Builder') ? 'Found' : 'NOT FOUND') . "\n";

// If Guzzle is available, test HTTP provider:
if (class_exists('GuzzleHttp\Client')) {
    $httpProvider = new GuzzleHttpProvider();
    $client = new ODataClient('https://services.odata.org/V4/TripPinService', null, $httpProvider);
    echo "✓ Basic client instantiation successful\n";
    
    // Test simple query (requires internet access)
    try {
        $people = $client->from('People')->get();
        echo "✓ Query execution successful, found " . $people->count() . " people\n";
    } catch (Exception $e) {
        echo "✗ Query failed: " . $e->getMessage() . "\n";
    }
} else {
    echo "⚠ Guzzle not available - HTTP provider testing skipped\n";
}
```

### Integration Test Scenarios
Run the existing test suite which covers:
- Entity set queries with various filters
- Pagination and cursor-based iteration  
- String contains/not-contains operations
- Skip token handling for large datasets
- Query building with where/orWhere clauses

## Build and Test Timing Expectations

- **Composer install (production only)**: 2-3 minutes  
- **Composer install (with dev dependencies)**: 10-20 minutes (NEVER CANCEL - network timeouts are common)
- **PHPUnit test suite**: 2-5 minutes (NEVER CANCEL - makes external HTTP calls)
- **Static analysis**: 1-2 minutes (if dependencies available)
- **Syntax validation** (`find src/ -name "*.php" -exec php -l {} \;`): ~1.5 seconds (34 files)
- **Autoload regeneration** (`composer dump-autoload`): <1 second

## Common Issues and Workarounds

### Dependency Installation Issues
- **Problem**: "Could not authenticate against github.com" during composer install
- **Solution**: This is expected due to GitHub API rate limits. Composer will fallback to source installations. Wait for completion.

- **Problem**: Network timeouts during dependency download
- **Solution**: Increase timeout values. Installation via source (git clone) takes longer but works reliably.

### Testing Issues  
- **Problem**: Test failures due to external service unavailability
- **Solution**: The test suite depends on services.odata.org being accessible. If tests fail due to network issues, this is environmental, not code-related.

### Development Dependencies
```bash
# If dev dependencies fail to install, try production-only first:
composer install --no-dev --no-interaction


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saintsystems/odata-client-php](https://github.com/saintsystems/odata-client-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
