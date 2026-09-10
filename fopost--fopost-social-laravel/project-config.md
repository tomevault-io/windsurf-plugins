---
trigger: always_on
description: This file provides guidance for Claude, Cursor, and other AI assistants working with the Fopost Laravel codebase.
---

# CLAUDE.md

This file provides guidance for Claude, Cursor, and other AI assistants working with the Fopost Laravel codebase.

## Project Overview

**Fopost Laravel** is the official Laravel integration for the Fopost social media publishing platform. It wraps [Fopost Core](https://github.com/fopost/fopost-social-core) with Laravel-idiomatic services: a service provider, facade, config file, and event bridging.

- **Repository:** `fopost/fopost-social-laravel`
- **Language:** PHP 8.1+
- **Framework:** Laravel 10.x, 11.x, 12.x
- **Core dependency:** `fopost/social-core` ^1.0
- **Namespace:** `Fopost\Social\Laravel\`
- **License:** MIT

## Relationship to Fopost Core

This package is a **thin wrapper**. All platform logic, formatting, HTTP transport, and content models live in `fopost-social-core`. This package provides:

1. **Service Provider** — Wires core classes into Laravel's container
2. **Facade** — Static access via `FopostSocial::telegram(...)`
3. **Configuration** — `config/fopost-social.php` populated from `.env`
4. **Event Bridge** — Routes core events through Laravel's event dispatcher

**Rule:** If a change involves platform behavior, API communication, or content formatting, it belongs in `fopost-social-core`. If it involves Laravel integration (DI, config, artisan, queues), it belongs here.

## Directory Structure

```
src/
├── Events/
│   └── LaravelEventDispatcher.php  # Bridges core events → Laravel events
├── Facades/
│   └── Fopost.php                # Facade for SendTo
├── FopostSocialServiceProvider.php     # Registers all services
└── SendTo.php                      # High-level publishing API

config/
└── fopost.php                    # Publishable config (credentials, platforms)

tests/
├── TestCase.php                    # Base test case (extends Orchestra Testbench)
├── Unit/
│   ├── SendToTest.php
│   └── ServiceProviderTest.php
├── Feature/
│   └── PublishingTest.php
└── fixtures/                       # Test fixture files

examples/
└── laravel-12/                     # Example Laravel 12 app
```

## Coding Standards

- **PHP version:** 8.1+ — use named arguments, enums, readonly properties, constructor promotion, union types where appropriate.
- **Strict types:** Every PHP file must start with `declare(strict_types=1);`.
- **Code style:** Follow PSR-12 coding standards.
- **Type hints:** All method parameters and return types must be fully typed. Use `mixed` only when truly necessary.
- **DocBlocks:** Use PHPDoc for complex parameter types (`@param array{key: type}`) and `@throws` annotations. Skip trivial docblocks where the type signature is self-explanatory.
- **Naming conventions:**
  - Classes: `PascalCase`
  - Methods/properties: `camelCase`
  - Constants: `UPPER_SNAKE_CASE`
  - Config keys: `snake_case`

## Service Container Bindings

The `FopostSocialServiceProvider` registers these bindings:

| Binding | Resolves To | Lifetime |
|---|---|---|
| `FopostConfig` | Config built from `config/fopost-social.php` | Singleton |
| `HttpClientInterface` | Core's cURL `HttpClient` (with optional proxy) | Singleton |
| `HashtagExtractor` | Core's hashtag extraction utility | Singleton |
| `CharacterTruncator` | Core's text truncation utility | Singleton |
| `TelegramFormatter`, `TwitterFormatter`, `FacebookFormatter` | Platform-specific formatters (using `HashtagExtractor` + `CharacterTruncator`) | Singleton |
| Platform instances | `TelegramPlatform`, `TwitterPlatform`, `FacebookPlatform`, `RedditPlatform`, `DiscordPlatform`, `SlackPlatform`, `InstagramPlatform`, `PinterestPlatform`, `WhatsAppPlatform`, `TumblrPlatform`, `LinkedInPlatform` | Singleton (only if credentials present) |
| `PlatformRegistry` | Registry of all active platforms | Singleton |
| `EventDispatcherInterface` | `LaravelEventDispatcher` | Singleton |
| `Publisher` | Core's `Publisher` with event dispatcher | Singleton |
| `'fopost-social'` / `SendTo` | `SendTo` instance | Singleton |

## Key Patterns

### SendTo API

`SendTo` provides shorthand methods for each platform:

```php
$sendTo->telegram('Hello!');           // Text message
$sendTo->twitter('Tweet!');            // Tweet
$sendTo->facebook('Post!', 'link', $data);  // Facebook post
$sendTo->toAll($post);                 // Publish to all platforms
```

Each method internally creates a `Post`, selects the platform from the registry, and delegates to `Publisher::publish()`.

### Event Bridging

`LaravelEventDispatcher` implements core's `EventDispatcherInterface` and forwards events through Laravel's `Illuminate\Events\Dispatcher`:

```php
// Core fires: PostPublished, PostFailed
// Laravel listeners can listen normally:
Event::listen(PostPublished::class, fn($e) => ...);
```

### Configuration Flow

```
.env variables → config/fopost-social.php → FopostConfig → Platform constructors
```

Only platforms with valid credentials are instantiated and registered.

## Build & Test Commands

```bash
# Install dependencies
composer install

# Run all tests
./vendor/bin/phpunit

# Run only unit tests
./vendor/bin/phpunit --testsuite=Unit

# Run only feature tests
./vendor/bin/phpunit --testsuite=Feature

# Shortcut
composer test
```

## Testing Approach


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fopost/fopost-social-laravel](https://github.com/fopost/fopost-social-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
