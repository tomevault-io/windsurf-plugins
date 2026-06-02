---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Nette Assets** is a PHP library for elegant asset management with automatic versioning, lazy loading, and support for multiple storage backends (filesystem, Vite). It provides seamless integration with Latte templates and the Nette framework.

**Key characteristics:**
- PHP 8.1+ requirement
- Zero-configuration default setup
- Registry pattern for managing multiple asset mappers
- Type-specific asset classes (Image, Script, Style, Audio, Video, Font, Generic)
- Lazy property loading for performance
- Vite integration with dev server auto-detection

## Essential Commands

### Testing

```bash
# Run all tests
vendor/bin/tester tests -s

# Run specific test file
php tests/Assets/ImageAsset.phpt

# Run tests in specific directory
vendor/bin/tester tests/Assets/ -s
```

### Static Analysis

```bash
# Run PHPStan
composer run phpstan

# Or directly
vendor/bin/phpstan analyse
```

### Code Standards

```bash
# Coding style is checked via GitHub Actions
# See .github/workflows/coding-style.yml
```

## Architecture

### Core Pattern: Mapper → Registry → Asset

The library uses a three-layer architecture:

1. **Mapper** - Resolves asset references to Asset objects
   - `FilesystemMapper` - Files from local filesystem with versioning
   - `ViteMapper` - Vite-generated assets with manifest.json support
   - Custom mappers can implement the simple `Mapper` interface

2. **Registry** - Central point managing multiple named mappers
   - Handles qualified references like `'images:logo.png'` or `['images', 'logo.png']`
   - Built-in LRU cache (max 100 entries) for resolved assets
   - Falls back to 'default' mapper when no prefix specified

3. **Asset** - Type-specific classes representing files
   - Base interface: `Asset` with `$url`, `$file`, `__toString()`
   - Specialized types: ImageAsset, ScriptAsset, StyleAsset, AudioAsset, VideoAsset, FontAsset, GenericAsset
   - EntryAsset for Vite entry points with dependencies
   - Properties are lazy-loaded using `LazyLoad` trait

### Asset Type Properties

Different asset types provide different readonly properties (all lazy-loaded):

- **ImageAsset**: `width`, `height`, `mimeType`, `alt`, `loading`
- **ScriptAsset**: `type`, `integrity`, `crossorigin`
- **StyleAsset**: `media`, `integrity`, `crossorigin`
- **AudioAsset**: `duration` (estimated for MP3 via `Helpers::guessMP3Duration()`), `mimeType`
- **VideoAsset**: `width`, `height`, `duration`, `poster`, `autoplay`, `mimeType`
- **FontAsset**: `mimeType`, `crossorigin` (always true for fonts)
- **GenericAsset**: `mimeType`
- **EntryAsset**: `imports` (array of StyleAsset), `preloads` (array of ScriptAsset), `crossorigin`

### Key Design Patterns

**Lazy Property Loading:**
- The `LazyLoad` trait defers expensive operations (reading dimensions, MIME types) until accessed
- Used across all asset types to keep asset creation fast
- Workaround for PHP < 8.4 lazy initialization

**Asset Type Detection:**
- `Helpers::createAssetFromUrl()` is the factory method
- Automatically selects asset class based on MIME type (from extension or explicit parameter)
- Extension-to-MIME mapping in `Helpers::ExtensionToMime`

**Mapper Options:**
- Every `Mapper::getAsset()` accepts `array $options` parameter
- `FilesystemMapper` supports `'version' => bool` option
- Custom mappers can define their own options for flexibility

**Vite Integration:**
- Automatic dev server detection via `.vite/vite-dev.json`
- Development mode: serves from Vite dev server with HMR
- Production mode: reads `manifest.json` for hashed filenames
- `EntryAsset` handles CSS imports and script preloads automatically

### Directory Structure

```
src/
├── Assets/               # Core library classes
│   ├── Asset.php        # Base interface
│   ├── *Asset.php       # Type-specific implementations
│   ├── Mapper.php       # Mapper interface
│   ├── FilesystemMapper.php
│   ├── ViteMapper.php
│   ├── Registry.php     # Central asset manager
│   ├── Helpers.php      # Static utilities
│   ├── LazyLoad.php     # Lazy loading trait
│   └── exceptions.php   # AssetNotFoundException
└── Bridges/
    ├── AssetsDI/        # Nette DI integration
    └── AssetsLatte/     # Latte template integration
```

### Bridge Architecture

**DI Extension (`Bridges/AssetsDI/DIExtension.php`):**
- Integrates with Nette DI container
- Parses NEON configuration for `assets:` section
- Creates Registry and mapper services
- Handles basePath/baseUrl resolution
- Supports dynamic parameters via Nette\Schema\DynamicParameter

**Latte Extension (`Bridges/AssetsLatte/LatteExtension.php`):**
- Provides `{asset}` tag and `n:asset` attribute
- Implements `asset()` and `tryAsset()` functions
- Implements `{preload}` tag for resource hints
- Node classes in `AssetsLatte/Nodes/` for AST compilation

## Testing Conventions

### Test File Structure

All tests use `.phpt` extension with Nette Tester's functional style:

```php
<?php
declare(strict_types=1);

use Tester\Assert;
require __DIR__ . '/../bootstrap.php';

test('Description of what is being tested', function () {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nette/assets](https://github.com/nette/assets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
