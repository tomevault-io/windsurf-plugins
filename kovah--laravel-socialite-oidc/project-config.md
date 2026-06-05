---
trigger: always_on
description: This is a PHP composer package that provides OpenID Connect (OIDC) authentication support for Laravel Socialite. It is a library package, not a standalone application.
---

# Laravel Socialite OIDC Provider

This is a PHP composer package that provides OpenID Connect (OIDC) authentication support for Laravel Socialite. It is a library package, not a standalone application.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Dependencies
- **CRITICAL**: `composer install` - Install PHP dependencies. Takes 2-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
  - **Network Issues**: If GitHub API timeouts occur, press Enter to skip OAuth token when prompted
  - **Alternative**: Use `COMPOSER_DISABLE_NETWORK=1 composer install --dry-run` to validate dependencies without network
- **Quick validation**: `composer check-platform-reqs` - Verify PHP 8.1+ and required extensions (takes < 1 second)

### Build and Validation
- **NO BUILD PROCESS** - This is a library package with no compilation step
- **Syntax validation**: Run `php -l src/Provider.php` and `php -l src/*.php` - validates all PHP files (takes < 1 second)
- **Package validation script**:
  ```bash
  php -r "
  echo 'Package Health Check:\n';
  echo 'PHP Version: ' . PHP_VERSION . (version_compare(PHP_VERSION, '8.1.0', '>=') ? ' ✓' : ' ✗') . '\n';
  echo 'JSON extension: ' . (extension_loaded('json') ? '✓' : '✗') . '\n';
  foreach (glob('src/*.php') as \$file) {
    \$output = []; exec('php -l ' . escapeshellarg(\$file) . ' 2>&1', \$output, \$return_var);
    echo basename(\$file) . ': ' . (\$return_var === 0 ? '✓' : '✗') . '\n';
  }
  "
  ```

### Testing
- **NO AUTOMATED TESTS** - This package has no test framework configured
- **Manual testing**: Requires integration with a Laravel application and OIDC provider
- **Integration test setup**:
  1. Create new Laravel app: `composer create-project laravel/laravel test-app`
  2. Add this package: `composer require kovah/laravel-socialite-oidc:dev-main`
  3. Configure OIDC provider in `config/services.php`
  4. Test authentication flow manually through browser

## Validation Scenarios

**CRITICAL**: Always test OIDC authentication flow after making changes:
1. **Provider Registration**: Verify `OIDCExtendSocialite` can register with Laravel's service container
2. **Configuration Loading**: Test provider loads OIDC configuration from `.well-known/openid-configuration` endpoint
3. **Authentication Flow**: Test redirect to OIDC provider → callback handling → user data extraction
4. **Error Handling**: Verify custom exceptions are thrown correctly for invalid states/tokens

**Manual validation steps**:
1. Always run syntax validation on changed PHP files
2. Test provider instantiation in Laravel context
3. Verify OIDC discovery endpoint integration
4. Test with real OIDC provider (Keycloak, Auth0, etc.)

## Repository Structure

### Core Implementation
- **`src/Provider.php`** - Main OIDC provider implementation (extends AbstractProvider)
  - Key methods: `redirect()`, `user()`, `getAccessTokenResponse()`, `getOpenIdConfig()`
  - Handles OIDC discovery, JWT validation, user data extraction
- **`src/OIDCExtendSocialite.php`** - Laravel service provider extension
  - Registers 'oidc' driver with Laravel Socialite

### Exception Classes
- **`src/ConfigurationFetchingException.php`** - OIDC discovery endpoint errors
- **`src/InvalidStateException.php`** - OAuth state validation failures  
- **`src/InvalidNonceException.php`** - OIDC nonce validation failures
- **`src/InvalidTokenException.php`** - JWT token validation errors
- **`src/InvalidCodeException.php`** - Authorization code validation errors
- **`src/EmptyEmailException.php`** - Missing email claim errors

### Key Configuration Points
- **Base URL**: OIDC provider base URL (excluding `.well-known/openid-configuration`)
- **Scopes**: Default `openid email profile`, customizable via config
- **PKCE**: Proof Key for Code Exchange support enabled by default
- **Nonce**: Used for replay attack prevention

## Common Development Tasks

### Adding New OIDC Claims
1. Modify `user()` method in `src/Provider.php` around line 280
2. Add claim mapping in the returned User object
3. Update documentation if claim affects public API

### Debugging OIDC Flow
1. **Configuration issues**: Check `getOpenIdConfig()` method for endpoint discovery
2. **Token validation**: Review JWT decoding in `decodeJWT()` method  
3. **User data**: Examine user info retrieval in `getUserByToken()` method
4. **State/nonce errors**: Check session handling in `redirect()` and `user()` methods

### Extending Provider Functionality
- Always extend `Provider` class rather than modifying directly
- Override specific methods: `getScopes()`, `getCodeFields()`, `mapUserToObject()`
- Add new exception classes following existing pattern (extend InvalidArgumentException)

## Integration with Laravel

### Service Provider Registration
**Laravel 11+**:
```php
Event::listen(function (\SocialiteProviders\Manager\SocialiteWasCalled $event) {
    $event->extendSocialite('oidc', \SocialiteProviders\OIDC\Provider::class);
});
```

**Laravel 10 and below**:
```php
protected $listen = [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kovah/laravel-socialite-oidc](https://github.com/Kovah/laravel-socialite-oidc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
