---
trigger: always_on
description: **Laravel Str Mixins** is a specialized Laravel package that extends `Illuminate\Support\Str` and `Illuminate\Support\Stringable` with additional functionality primarily designed for Japanese text processing. The package provides enhanced string manipulation methods that work better with multibyte characters and Japanese-specific text formatting needs.
---

# GitHub Copilot Onboarding Guide for Laravel Str Mixins

## Repository Overview

**Laravel Str Mixins** is a specialized Laravel package that extends `Illuminate\Support\Str` and `Illuminate\Support\Stringable` with additional functionality primarily designed for Japanese text processing. The package provides enhanced string manipulation methods that work better with multibyte characters and Japanese-specific text formatting needs.

### Primary Purpose
- Extend Laravel's built-in string helpers with Japanese-friendly alternatives
- Provide proper multibyte character handling for text processing
- Offer both static Str methods and fluent Stringable chain methods

## Key Features

### 1. Text Wrapping (`textwrap`)
- **Purpose**: Force text wrapping at specified character width
- **Why needed**: Laravel's `wordWrap()` doesn't work properly with Japanese text
- **Usage**: `Str::textwrap('text', 10)` or `Str::of('text')->textwrap(10)`

### 2. Japanese Character Conversion (`kana`)
- **Purpose**: Convert between different Japanese character types (hiragana, katakana, half-width, full-width)
- **Implementation**: Wrapper around PHP's `mb_convert_kana()` function
- **Usage**: `Str::kana('text', 'KVa')` or `Str::of('text')->kana('KVa')`

### 3. Character-Count Truncation (`truncate`)
- **Purpose**: Truncate text by character count rather than display width
- **Why needed**: Laravel's `limit()` counts half-width as 1 and full-width as 2, which isn't ideal for Japanese
- **Usage**: `Str::truncate('text', 10, '...')` or `Str::of('text')->truncate(10, '...')`

## Architecture Overview

### Package Structure
```
src/
├── StrMixinsServiceProvider.php    # Service provider that registers mixins
├── Str/                           # Static Str macro implementations
│   ├── Kana.php                   # Japanese character conversion
│   ├── TextWrap.php               # Text wrapping functionality
│   └── Truncate.php               # Character-count truncation
└── Stringable/                    # Fluent Stringable mixins
    └── Japanese.php               # Chainable Japanese text methods
```

### Design Patterns

#### 1. Invokable Classes for Str Macros
```php
class TextWrap
{
    public function __invoke(?string $str, int $width = 10, string $break = PHP_EOL): string
    {
        return implode($break, mb_str_split($str ?? '', $width));
    }
}
```

#### 2. Closure-returning Methods for Stringable Mixins
```php
public function textwrap(): Closure
{
    return function (int $width = 10, string $break = PHP_EOL): Stringable {
        return new Stringable(Str::textwrap($this->value, $width, $break));
    };
}
```

## Development Setup

### Requirements
- PHP >= 8.3
- Laravel >= 12.0
- ext-mbstring (for multibyte string functions)

### Installation & Setup
```bash
# Clone and setup
git clone https://github.com/invokable/laravel-str-mixins.git
cd laravel-str-mixins
composer install

# Run tests
./vendor/bin/phpunit

# Check code style
./vendor/bin/pint --test

# Fix code style
./vendor/bin/pint
```

### Testing Environment
- Uses Orchestra Testbench for Laravel package testing
- PHPUnit for unit tests
- All tests should pass and maintain 100% coverage when possible

## Code Style & Conventions

### PHP Standards
- **Strict types**: Always use `declare(strict_types=1);`
- **Laravel Pint**: Follow Laravel coding standards with pint.json configuration
- **PSR-4 autoloading**: `Revolution\Laravel\Mixins` namespace

### Key Conventions
1. **Null handling**: All methods should handle null input gracefully (`$str ?? ''`)
2. **UTF-8 encoding**: Always specify encoding for multibyte functions
3. **Method signatures**: Match Laravel's existing patterns where possible
4. **Return types**: Always specify return types explicitly

### Example Code Pattern
```php
<?php

declare(strict_types=1);

namespace Revolution\Laravel\Mixins\Str;

class ExampleMethod
{
    public function __invoke(?string $str, int $param = 10): string
    {
        $str = $str ?? '';
        
        // Your multibyte-safe implementation here
        return mb_substr($str, 0, $param, 'UTF-8');
    }
}
```

## Testing Patterns

### Test Structure
```php
class StrTest extends TestCase
{
    public function test_method_name()
    {
        // Test normal case
        $this->assertSame('expected', Str::method('input'));
        
        // Test Japanese text
        $this->assertSame('あいう', Str::method('あいうえお', 3));
        
        // Test null handling
        $this->assertSame('', Str::method(null));
    }
}
```

### Important Test Cases
1. **Null input handling**: Every method should handle null gracefully
2. **Japanese text**: Test with hiragana, katakana, and kanji
3. **Mixed content**: Test with mixed ASCII and Japanese characters
4. **Edge cases**: Empty strings, very long strings, special characters

## Contributing Guidelines

### Adding New Features

1. **For Str macros**:
   - Create invokable class in `src/Str/`
   - Register in `StrMixinsServiceProvider::boot()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/invokable) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
