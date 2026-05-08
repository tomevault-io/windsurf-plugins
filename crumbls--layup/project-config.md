---
trigger: always_on
description: Generates Alpine.js directives for entrance animations. Returns an empty string if no animation is configured. Use `{!! !!}` (unescaped) because the output contains HTML attributes.
---

# Layup Widget Development Guide for AI Agents

This document is a complete reference for AI coding agents (Claude Code, Cursor, Copilot, etc.) to create Layup widgets without error. Follow every rule precisely. Deviations cause runtime failures, missing form fields, or broken rendering.

## Architecture Overview

A Layup widget consists of exactly three files:

1. **PHP class** -- extends `Crumbls\Layup\View\BaseWidget`, defines form fields, defaults, preview, and metadata
2. **Blade view** -- renders the widget's frontend HTML using the `$data` array
3. **Test file** (optional) -- Pest test using `LayupAssertions` trait

The PHP class is a static configuration object. All public methods are `static`. The class is never instantiated by the developer -- `BaseWidget::make($data)` handles construction internally. The Blade view receives `$data` (associative array) and `$children` (array of child view components, usually empty for widgets).

## File Locations

| File | Path | Convention |
|------|------|------------|
| PHP class | `app/Layup/Widgets/{ClassName}Widget.php` | PascalCase, must end in `Widget` |
| Blade view | `resources/views/components/layup/{type}.blade.php` | kebab-case, matches `getType()` |
| Test file | `tests/Unit/Layup/{ClassName}WidgetTest.php` | Matches class name |

For package development (inside crumbls/layup itself):

| File | Path |
|------|------|
| PHP class | `src/View/{ClassName}Widget.php` |
| Blade view | `resources/views/components/{type}.blade.php` |
| Test file | `tests/Unit/{ClassName}WidgetTest.php` |

## The Critical Rule: Field Name Alignment

**Every key in `getDefaultData()` must exactly match a field name in `getContentFormSchema()`, and vice versa.** This is the most common source of errors. The builder stores data using the field names from the form schema. The Blade view accesses data using the keys from `getDefaultData()`. If these do not align, fields silently lose their values or Blade views throw `Undefined array key` errors.

```php
// CORRECT -- field names match default data keys
public static function getContentFormSchema(): array
{
    return [
        TextInput::make('title'),     // field name: 'title'
        TextInput::make('subtitle'),  // field name: 'subtitle'
    ];
}

public static function getDefaultData(): array
{
    return [
        'title' => '',       // matches 'title'
        'subtitle' => '',    // matches 'subtitle'
    ];
}

// WRONG -- mismatch causes silent data loss
public static function getContentFormSchema(): array
{
    return [
        TextInput::make('heading'),  // field name: 'heading'
    ];
}

public static function getDefaultData(): array
{
    return [
        'title' => '',  // WRONG -- does not match 'heading'
    ];
}
```

For `Repeater` fields, the default must be an array of associative arrays matching the repeater's child schema:

```php
Repeater::make('items')
    ->schema([
        TextInput::make('title'),
        RichEditor::make('content'),
    ])

// Default:
'items' => [
    ['title' => 'Item 1', 'content' => ''],
    ['title' => 'Item 2', 'content' => ''],
]
```

## PHP Class Template

Every widget class follows this exact structure. Do not add constructors, instance properties, or non-static methods other than `render()`.

```php
<?php

declare(strict_types=1);

namespace App\Layup\Widgets;

use Crumbls\Layup\View\BaseWidget;
use Crumbls\Layup\Support\WidgetContext;
use Filament\Forms\Components\TextInput;

class ExampleWidget extends BaseWidget
{
    public static function getType(): string
    {
        return 'example';
    }

    public static function getLabel(): string
    {
        return 'Example Widget';
    }

    public static function getIcon(): string
    {
        return 'heroicon-o-cube';
    }

    public static function getCategory(): string
    {
        return 'content';
    }

    public static function getContentFormSchema(): array
    {
        return [
            TextInput::make('title')
                ->label('Title')
                ->required(),
        ];
    }

    public static function getDefaultData(): array
    {
        return [
            'title' => '',
        ];
    }

    public static function getPreview(array $data): string
    {
        return $data['title'] ?? '(empty)';
    }
}
```

## Required Static Methods

### getType(): string

Returns a unique kebab-case identifier. This value:
- Is stored in the JSON content structure
- Maps to the Blade view path: `components.layup.{type}` (for app widgets) or `layup::components.{type}` (for package widgets)
- Must be unique across all registered widgets

```php
public static function getType(): string
{
    return 'pricing-card';
}
```

### getLabel(): string

Human-readable name shown in the widget picker. For built-in widgets, use translation keys. For app widgets, a plain string is fine.

```php
// Built-in widget:
public static function getLabel(): string
{
    return __('layup::widgets.labels.pricing-card');
}

// App widget:
public static function getLabel(): string
{
    return 'Pricing Card';
}
```

### getIcon(): string


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crumbls/layup](https://github.com/Crumbls/layup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
