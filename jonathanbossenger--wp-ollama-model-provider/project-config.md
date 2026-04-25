---
trigger: always_on
description: This is a WordPress plugin that provides Ollama AI model support (local and cloud) for the WordPress AI Client. It enables WordPress to use Ollama models by acting as a provider for the [WordPress AI Client](https://github.com/WordPress/wordpress-ai-client).
---

# WP Ollama Model Provider - Copilot Instructions

## Project Overview

This is a WordPress plugin that provides Ollama AI model support (local and cloud) for the WordPress AI Client. It enables WordPress to use Ollama models by acting as a provider for the [WordPress AI Client](https://github.com/WordPress/wordpress-ai-client).

**Key Technologies:**
- PHP 8.0+ with strict types
- WordPress 6.0+
- Composer (PSR-4 autoloading)
- WordPress AI Client library (`wordpress/wp-ai-client`)
- Ollama (local AI model server or cloud)

## Build, Lint, and Test Commands

```bash
# Install dependencies (required after clone)
composer install

# Lint (WordPress Coding Standards)
composer run phpcs

# Auto-fix coding standards issues
composer run phpcbf

# Run all tests
composer run test
# Or directly:
vendor/bin/phpunit

# Run specific test file
vendor/bin/phpunit tests/SomeTest.php
```

**Note:** Tests currently only have a bootstrap file. The project expects Ollama to be running locally for integration testing.

## Architecture

### Provider Pattern

The plugin uses a provider pattern to integrate with WordPress AI Client:

```
wp-ollama-model-provider.php (entry point)
    ↓
Registers OllamaProvider with AI Client registry
    ↓
includes/Providers/Ollama/
    ├── OllamaProvider.php           - Main provider class (extends AbstractApiProvider)
    ├── OllamaTextGenerationModel.php - Text generation implementation
    ├── OllamaModelMetadataDirectory.php - Model metadata/discovery
    ├── NoAuthRequestAuthentication.php - No-auth handler (local = no API keys)
    └── ApiKeyRequestAuthentication.php - API key handler (cloud mode)
```

**Key Flow:**
1. Plugin init hook (`wp_ollama_model_provider_init`) registers the provider
2. Provider registers with `WordPress\AiClient\AiClient::defaultRegistry()`
3. Authentication is set based on deployment mode (local or cloud)
4. Settings page allows users to select which Ollama model to use
5. Other plugins query selected model via public API functions

### Settings Architecture

Settings are stored as WordPress options:
- `wp_ollama_model_provider_ollama_deployment_mode` - Deployment mode (local or cloud)
- `wp_ollama_model_provider_ollama_api_key` - API key for cloud mode
- `wp_ollama_model_provider_ollama_model` - Selected model ID
- Model list cached in transient `wp_ollama_model_provider_ollama_models` (5 min TTL)

Admin settings page: `Settings > Ollama AI Models`

### Public API

Three public functions for plugin integration:
```php
wp_ollama_model_provider_is_provider_registered( 'ollama' )  // bool
wp_ollama_model_provider_has_settings_page()                 // bool
wp_ollama_model_provider_get_selected_model( 'ollama' )      // string (model ID)
```

## Key Conventions

### WordPress Coding Standards

Follows [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/):
- WordPress naming: `wp_ollama_model_provider_function_name()`
- Prefixes: All functions/hooks/options use `wp_ollama_model_provider_` prefix
- Strict typing: All class files use `declare(strict_types=1);`
- Namespaces: PSR-4 under `WpOllamaModelProvider\`

### Filters

Three key filters for customization:
```php
// Change Ollama server URL (default: http://localhost:11434)
apply_filters( 'wp_ai_client_ollama_base_url', 'http://localhost:11434' )

// Change Ollama Cloud URL (default: https://ollama.com)
apply_filters( 'wp_ai_client_ollama_cloud_base_url', 'https://ollama.com' )

// Adjust request timeout for slower models
apply_filters( 'wp_ai_client_default_request_timeout', $timeout )
```

### Error Handling

- Use `WP_Error` for user-facing errors (model connection, API errors)
- Use `Exception` for integration failures (caught and logged when `WP_DEBUG` is true)
- Model discovery failures return `WP_Error`, not exceptions

### Caching Strategy

Model list is cached as a transient for 5 minutes to reduce API calls to Ollama:
```php
get_transient( 'wp_ollama_model_provider_ollama_models' )
set_transient( 'wp_ollama_model_provider_ollama_models', $models, 5 * MINUTE_IN_SECONDS )
delete_transient( 'wp_ollama_model_provider_ollama_models' ) // On refresh
```

### Plugin Constants

```php
WP_OLLAMA_MODEL_PROVIDER_VERSION  // Plugin version
WP_OLLAMA_MODEL_PROVIDER_PATH     // Plugin directory path
WP_OLLAMA_MODEL_PROVIDER_URL      // Plugin directory URL
```

## Provider Extensibility

The plugin is designed to support multiple providers in the future:
- Current: Ollama only (local and cloud)
- Future: LocalAI, LM Studio, etc.

When adding new providers:
1. Create `includes/Providers/{ProviderName}/` directory
2. Implement provider class extending `AbstractApiProvider`
3. Register in main plugin file
4. Add settings section for provider-specific options
5. Update public API to support new provider slug

## WordPress Integration

### Installation Methods

1. **Composer** (recommended): `composer require jonathanbossenger/wp-ollama-model-provider`
   - Works with Bedrock: installs to `web/app/plugins/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonathanbossenger/wp-ollama-model-provider](https://github.com/jonathanbossenger/wp-ollama-model-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
