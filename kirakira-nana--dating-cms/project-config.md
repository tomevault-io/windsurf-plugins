---
trigger: always_on
description: This file applies to the entire repository. More specific `AGENTS.md` files, if
---

# pH7Builder Agent Guide

## Scope and sources of truth

This file applies to the entire repository. More specific `AGENTS.md` files, if
added later, override it only within their directories.

Before changing code, read the relevant nearby files and
[`CONTRIBUTING.md`](CONTRIBUTING.md). Treat the current implementation,
`.editorconfig`, `.php-cs-fixer.dist.php`, `phpcs.xml.dist`,
`phpstan.neon.dist`, and `phpunit.xml.dist` as the executable sources of truth.
Keep changes focused and preserve unrelated work already present in the
worktree.

pH7Builder targets PHP 8.2 or newer and combines a modular MVC application with
the project-specific pH7Framework and PH7Tpl template engine.

## Repository map

- `_protected/app/system/modules/`: bundled application modules. A module
  commonly contains `controllers/`, `models/`, `forms/`,
  `forms/processing/`, `views/`, `config/`, `lang/`, `inc/`, and `assets/`.
- `_protected/app/modules/`: optional/custom modules. Use the `helloworld`
  module as the minimal structural example.
- `_protected/app/system/core/`: shared application controllers, models,
  forms, classes, and assets used across modules.
- `_protected/app/includes/`: application base classes, helpers, and the
  application autoloader.
- `_protected/app/configs/routes/`: language-aware XML routes. Module routing
  may select either `system/modules` or `modules`.
- `_protected/framework/`: pH7Framework. It uses its own autoloader and filename
  suffixes rather than Composer PSR-4 for most framework classes.
- `_protected/app/system/modules/*/views/<theme>/tpl/`: module PH7Tpl
  templates.
- `templates/themes/`: global page layouts and public theme assets.
- `templates/system/modules/`: public CSS, JavaScript, images, and configuration
  belonging to system-module themes.
- `static/`: shared browser assets and vendored browser libraries.
- `data/`: public runtime/user media. Most runtime content is ignored; commit
  only intentional default assets or placeholder files.
- `_protected/data/`: private cache, logs, backups, temporary files, fonts, and
  bundled backgrounds. Do not commit generated runtime files.
- `_install/`: installer application and the canonical new-install SQL schemas.
- `_repository/upgrade/`: versioned upgrade migrations. Do not rewrite historic
  migrations; add a migration for the intended upgrade path.
- `_tests/Unit/`: PHPUnit tests. Test namespaces and directories should mirror
  the source area as closely as the existing suite permits.
- `_tools/`: project maintenance, CLI, and local preview tools.

Put a change in the narrowest owning layer. Keep HTTP orchestration in
controllers, persistence in models, reusable application behavior in
`system/core` or `includes`, and generic infrastructure in the framework.
Generate application URLs with `Uri::get()` and respect the XML routing model
instead of hardcoding route strings.

## Coding conventions

Follow the surrounding file when legacy compatibility constrains a change, but
use the conventions below for new or substantially rewritten code.

### Classes, interfaces, traits, methods, and constants

- Use `UpperCamelCase` alphanumeric names for classes, interfaces, and traits.
- Prefer adjective interface names ending in `-able` or `-ible` when natural:
  `Controllable`, `Configurable`, `Hashable`, `Readable`, `Serializable`.
- Use `camelCase` alphanumeric method names.
- Use `ALL_CAPS_WITH_UNDERSCORES` for constants.
- Use the least visibility that supports the design. Public controller actions
  and public APIs are expected; avoid public mutable properties.
- Add parameter, property, and return types where they are accurate and
  compatible with the inheritance hierarchy. The project naming convention
  complements native PHP types; it does not replace them.
- Do not introduce compatibility aliases, duplicate implementations, or dead
  fallback code without a demonstrated caller.

Example:

```php
final class MyClass
{
    private const EXPECTED_VALUE = 'abcd';

    public function normalizeValue(string $sValue): string
    {
        return $sValue === self::EXPECTED_VALUE
            ? self::EXPECTED_VALUE
            : 'zyxw';
    }
}
```

### Framework and application filenames

- In `_protected/framework`, classes end in `.class.php`, traits in
  `.trait.php`, and interfaces in `.interface.php`. The filename and declared
  symbol must match exactly.
- Application classes under `_protected/app` normally use plain `.php`
  filenames, following the module's established structure.
- PHPUnit test classes end in `Test.php`.
- Do not rename framework files to ordinary `.php`; the framework autoloader
  explicitly resolves the project suffixes.

### Variables and data names

Variables use camelCase with a leading type indicator. Keep the prefix accurate
when a value changes type:

- `a`: array
- `i`: integer
- `f`: float/double
- `b`: boolean
- `c`: one character
- `s`: string
- `by`: byte, in legacy/low-level contexts
- `r`: resource
- `o`: object
- `m`: mixed

Examples include `$aUsers`, `$iProfileId`, `$fPrice`, `$bEnabled`, `$sUsername`,
`$oUser`, and `$mValue`.

Use `lowercase_with_underscores` for new global functions, global/session/cookie

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirakira-nana/Dating-CMS](https://github.com/kirakira-nana/Dating-CMS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
