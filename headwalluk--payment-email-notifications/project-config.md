---
trigger: always_on
description: **Purpose:** Core coding standards and patterns for WordPress plugin development
---

# WordPress Plugin Development Guidelines

**Version:** 1.6.0  
**Purpose:** Core coding standards and patterns for WordPress plugin development  
**Portability:** Copy this file to any WordPress plugin project  
**Last Updated:** 31 January 2026

**📚 Detailed Patterns:** See [`dev-notes/patterns/README.md`](../dev-notes/patterns/README.md) for implementation details

---

## Core Principles

1. **Follow WordPress Coding Standards** - Verify with phpcs/phpcbf
2. **Use Modern PHP** - PHP 8.0+ features (type hints, union types, nullable types)
3. **Write Self-Documenting Code** - Clear naming reduces need for comments
4. **Security First** - Always sanitize input, escape output, verify nonces
5. **Keep It Simple** - Favor clarity over cleverness

---

## Protected Directories

### Power Plugins Licence Controller (`pwpl/`)

**Rule:** If a `pwpl/` directory exists in the plugin's base directory, **DO NOT ALTER** any files within it.

**Purpose:** Contains the Power Plugins licence management system for commercial distribution.

**Integration:** The main plugin file must include the licence controller early:

```php
// Power Plugins licence control.
require_once __DIR__ . '/pwpl/pwpl.php';
```

**Important:**
- Never modify, refactor, or "improve" code in `pwpl/`
- Do not update coding standards in this directory
- Do not include `pwpl/` files in PHPCS checks
- Treat this directory as a sealed third-party dependency

---

## File Structure & Naming

### Directory Organization

```
example-plugin/
├── example-plugin.php        # Main plugin file
├── constants.php             # Plugin constants
├── functions-private.php     # Private/internal functions
├── phpcs.xml                 # Code standards configuration
├── includes/                 # Core classes
│   ├── class-plugin.php
│   ├── class-settings.php
│   └── class-admin-hooks.php
├── admin-templates/          # Admin template files
├── templates/                # Public template files
├── assets/
│   ├── admin/               # Admin CSS/JS
│   └── public/              # Public CSS/JS
├── dev-notes/               # Development documentation
└── languages/               # Translation files
```

### File Naming Conventions

- **Class files:** \`class-{classname}.php\` (lowercase with hyphens)
- **Template files:** \`{template-name}.php\` (lowercase with hyphens)
- **Asset files:** \`{descriptive-name}.{ext}\` (lowercase with hyphens)

---

## PHP Standards

### Namespaces & Classes

```php
namespace Example_Plugin;

defined( 'ABSPATH' ) || die();

class Settings {
    // No prefix needed - namespace handles it
}
```

**❌ Do NOT use \`declare(strict_types=1);\`**

WordPress and WooCommerce do not use strict types. Using it causes type errors when WordPress passes strings where you expect ints, breaks hook/filter interoperability, and fails with WooCommerce CRUD methods.

### Class Organization

**Order:**
1. Properties (public → protected → private)
2. \`__construct()\`
3. Methods (public → protected → private)

### Type Hints & Documentation

```php
/**
 * Brief description.
 *
 * @since 1.0.0
 *
 * @param string $param1 Description.
 * @param int    $param2 Description.
 *
 * @return array<string, mixed> Description.
 */
public function example_function( string $param1, int $param2 ): array {
    // Implementation
}
```

### Function Structure - Single-Entry Single-Exit (SESE)

**Rule:** Functions should have one return statement at the end.

**Why:** Easier debugging and fault tracing - you can see all logic paths that affect the final value.

```php
// ✅ Good - Single exit point
function get_value(): string {
    $result = null;
    
    if ( condition_a() ) {
        $result = 'value_a';
    }
    
    if ( is_null( $result ) && condition_b() ) {
        $result = 'value_b';
    }
    
    if ( is_null( $result ) ) {
        $result = 'default';
    }
    
    return $result;
}

// ❌ Avoid - Multiple exit points obscure which path executed
function get_value(): string {
    if ( condition_a() ) {
        return 'value_a';
    }
    
    if ( condition_b() ) {
        return 'value_b';
    }
    
    return 'default';
}
```

### Constants & Magic Values

**Rule:** All magic strings and numbers must be defined as constants in \`constants.php\`.

**Exception:** Translatable text strings use \`__()\` or \`_e()\` directly.

```php
// constants.php
namespace Example_Plugin;

// Default values - prefix with DEF_
const DEF_CHECKOUT_ERROR_MESSAGE = 'Item no longer available.';

// wp_options keys - prefix with OPT_
const OPT_TIME_FORMAT = 'example_plugin_time_format';

// Magic numbers
const MAX_SLOTS_PER_DAY = 20;
```

### Boolean Options - Handle Multiple Formats

Use \`filter_var()\` with \`FILTER_VALIDATE_BOOLEAN\` to handle all boolean formats:

```php
// ✅ Good - handles '1', 'yes', 'on', true, etc.
$enabled = (bool) filter_var( get_option( OPT_ENABLED, false ), FILTER_VALIDATE_BOOLEAN );

// ❌ Bad - only checks specific string
if ( 'yes' === get_option( OPT_ENABLED ) ) { }
```

---

## Date/Time Handling

**Rule:** Store date/time as human-readable strings in format \`Y-m-d H:i:s T\`, NOT Unix timestamps.

```php
/**
 * Get current timestamp in human-readable format.
 */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [headwalluk/payment-email-notifications](https://github.com/headwalluk/payment-email-notifications) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
