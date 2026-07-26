---
trigger: always_on
description: Laravel OpenTelemetry is a PHP package that provides comprehensive OpenTelemetry integration for Laravel applications, supporting traces, metrics, and logs with various exporters (OTLP, Zipkin, Console). **This is a library/package, not a standalone application** - it integrates into Laravel projects.
---

# Laravel OpenTelemetry Package

Laravel OpenTelemetry is a PHP package that provides comprehensive OpenTelemetry integration for Laravel applications, supporting traces, metrics, and logs with various exporters (OTLP, Zipkin, Console). **This is a library/package, not a standalone application** - it integrates into Laravel projects.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup
- Install Docker and Docker Compose for containerized development
- The project uses PHP 8.1+ and requires Redis for testing
- GRPC PHP extension is optional but recommended for full OTLP gRPC functionality

### Bootstrap, Build, and Test the Repository

**Docker-based Development (Recommended):**
- `make build` -- Builds Docker image with proper UID/GID permissions. Takes 3-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
- `make start` -- Starts containers in background (app + Redis). Takes 30-60 seconds.
- `make shell` -- Opens interactive shell in app container. Always run `make start` first.
- `make test` -- Runs full test suite via Composer in container. Takes 2-3 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
- `make lint` -- Runs code quality tools (Pint + PHPStan). Takes 1-2 minutes. NEVER CANCEL. Set timeout to 5+ minutes.
- `make stop` -- Stops and removes containers

**Local Development (Alternative):**
- `composer install --ignore-platform-req=ext-grpc` -- Install dependencies, ignoring GRPC if not available. Takes 3-6 minutes. NEVER CANCEL. Set timeout to 15+ minutes.
- `composer test` -- Run tests via Composer script (uses Pest framework)
- `composer run lint` -- Run code quality tools (Pint + PHPStan)
- Direct binaries available after full install: `vendor/bin/pest`, `vendor/bin/pint`, `vendor/bin/phpstan`

### Docker Build Troubleshooting
- **Common failure**: `make build` fails with "No releases available for package pecl.php.net/grpc" due to PECL network connectivity
- **Error message**: `install-php-extensions grpc` returns exit code 1 with PECL connection failures
- **Solution**: The GRPC extension is optional for basic development and testing
- **Alternative**: Use local development with `composer install --ignore-platform-req=ext-grpc`
- **CI Environment**: GitHub Actions and production environments have GRPC extension pre-installed
- **Network dependency**: Docker build success depends on external PECL repository availability

### Common Installation Issues
- **Network timeouts**: Composer may timeout downloading packages from GitHub. Increase timeout and retry.
- **Missing GRPC**: Use `--ignore-platform-req=ext-grpc` flag with Composer commands
- **Memory limits**: Large dependency tree may require increasing PHP memory limit
- **Permission issues with Docker**: Ensure proper UID/GID mapping in `make build`

## Validation

### Quick Setup Validation
After cloning the repository, verify the environment with these commands:
```bash
# Check basic tools
php --version  # Should be 8.1+
composer --version
docker --version

# Test basic setup
composer install --no-dev --ignore-platform-req=ext-grpc  # ~2 minutes
composer run-script --list  # Should show: test, test-coverage, lint

# Verify package structure
ls -la src/Instrumentation/  # Should show PHP instrumentation classes
ls -la config/opentelemetry.php  # Package configuration
```

### Manual Testing Scenarios
Since this is a Laravel package (not a standalone application), testing involves:

1. **Unit Test Validation**: Always run the full test suite to verify functionality
   - `make test` or `composer test`
   - Test suite covers all instrumentation classes, SDK components, and integration points
   - Uses Pest framework with in-memory exporters for isolation
   - Helper functions: `getRecordedSpans()`, `getRecordedMetrics()`, `getRecordedLogs()`

2. **Static Analysis Validation**: Ensure code quality standards
   - `make lint` or `vendor/bin/phpstan`
   - PHPStan runs at level 8 with strict checking

3. **Code Style Validation**: Maintain consistent formatting
   - `vendor/bin/pint` for automatic code formatting
   - Laravel Pint configuration is included in the repository

### Package Functionality Testing
Since this is an instrumentation library, you cannot "run" it standalone. Instead validate by:
- **Trace Testing**: Check spans are recorded for HTTP, DB, Redis, Queue operations
- **Metrics Testing**: Verify custom meters work with Counter, Histogram, Gauge
- **Logs Testing**: Confirm OpenTelemetry log integration captures Laravel logs
- **Integration Testing**: Test with real Laravel application using the package

### Example Test Commands
```bash
# Run specific test suites
composer test -- --filter=Tracer
composer test -- --filter=HttpServerInstrumentation
composer test -- tests/Instrumentation/

# Check specific functionality
php artisan route:list  # Not applicable - this is a package, not app

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keepsuit/laravel-opentelemetry](https://github.com/keepsuit/laravel-opentelemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
