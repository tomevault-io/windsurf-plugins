---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TallStackUI is a suite of Blade components for Laravel TALL Stack applications (Tailwind CSS, Alpine.js, Laravel, Livewire). It provides 40+ reusable components for building modern web interfaces. Our current goal is prepare the release of v3.0.  Check it out it to understand more about the components: https://context7.com/websites/tallstackui_v2/llms.txt?tokens=10000

**Stack:**

- PHP 8.1+
- PestPHP 4
- Laravel 10/11/12/13
- Livewire 3.5+
- Tailwind CSS 4
- Alpine.js 3
- Vite 7

## Essential Commands

```bash
# Build & Development
npm run build              # Build JS + Tailwind CSS
npm run dev                # Watch mode

# Testing
composer test                    # Run all Pest tests
composer test:feature --parallel # Feature tests only (parallel)
composer test:browser            # Browser/Dusk tests only
composer type                    # Type coverage check

# Code Quality
./vendor/bin/pint --parallel     # Format PHP
npm run lint:fix                 # Fix ESLint issues
npm run format                   # Format JS with Prettier
composer format                  # Run all formatters (pint + eslint + prettier)

# Static Analysis
composer analyse           # PHPStan level 5 (via larastan)
composer rector            # Rector dry-run

# CI Pipeline
composer ci                # Full CI: pint, feature tests, browser tests
composer ci:analyse        # Pint + feature tests only

# Utilities
composer bench             # Orchestra Testbench
composer test:browser:setup # Update ChromeDriver for Dusk
```

## Development Workflow

After completing any code changes (PHP, JS, or CSS), always run:

```bash
npm run build
```

This ensures the built assets in `dist/` are updated and reflect your changes.

## Testing Rules

- **Browser tests**: NEVER run with `--parallel`. Always run with `--filter` to target specific test classes or methods. Example: `./vendor/bin/pest --filter="can_render_with_highlighted"`
- **Feature tests**: CAN run with `--parallel`. Example: `composer test:feature --parallel`
- Never run browser tests in parallel mode.

## Architecture

### Component Structure

Components live in `src/View/Components/` and extend `TallStackUiComponent`. Each component has:

- A PHP class defining properties and customization
- A Blade template in `src/resources/views/components/`
- Optional color classes in `src/Support/Colors/Components/`

**Component Organization (70+ classes):**

| Directory      | Components                                                                                                                                                                                                            |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Root           | Alert, Avatar, Badge, Banner, Boolean, Card, Carousel, Clipboard, Environment, Errors, Floating, Icon, KeyValue, Link, Loading, Modal, Rating, Reaction, Signature, Slide, Stats, Table, ThemeSwitch, Tooltip, Upload |
| `Button/`      | Button, Circle                                                                                                                                                                                                        |
| `Dropdown/`    | Dropdown, Items, Submenu                                                                                                                                                                                              |
| `Form/`        | Checkbox, Color, Currency, Date, Error, Hint, Input, Label, Number, Password, Pin, Radio, Range, Tag, Textarea, Time, Toggle, Upload                                                                                  |
| `Form/Select/` | Native, Styled                                                                                                                                                                                                        |
| `Interaction/` | Dialog, Toast                                                                                                                                                                                                         |
| `Layout/`      | Layout, Header, SideBar (Item, Separator)                                                                                                                                                                             |
| `Progress/`    | Progress, Circle                                                                                                                                                                                                      |
| `Step/`        | Step, Items                                                                                                                                                                                                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tallstackui/tallstackui](https://github.com/tallstackui/tallstackui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
