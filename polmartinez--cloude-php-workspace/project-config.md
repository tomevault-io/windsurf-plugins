---
trigger: always_on
description: How to write efficient, idiomatic code for the Cloude PHP micro-framework (cloude/framework, `Cloude\` namespace). Use this WHENEVER the project includes `cloude/framework` in its `composer.json`, imports classes from the `Cloude\` namespace (Router, Bootstrap, View, Input, Http\Response, Markdown, Data\JsonRepository, Data\MarkdownRepository, Mcp\Server, TaskRunner, Cli, Logger, Collection, Arr, Str, Format, JsonFile, JsonSchema, Config, EventLog, AssetUrl, Cache, ErrorHandler), or whenever the
---


# Cloude Framework — Quick guide for writing efficient code

Cloude is a minimalist PHP 8.4+ micro-framework: **one class per file, no magic, no container, no DSL, no full ORM**. PSR-4 (`Cloude\`), PSR-12 / PER-CS 2.0, `declare(strict_types=1)` in every file. Runtime dependencies: zero (only `ext-intl` recommended for non-Latin slug transliteration).

> **Rule #1 before writing any code**: if you find yourself reaching for "a service", "a container", "a middleware stack" or "a factory" — stop. Cloude doesn't have them on purpose. Solve it with the function or static class that already exists.

## When NOT to use this skill

- Laravel / Symfony / CodeIgniter / CakePHP project → use that framework's skill.
- WordPress or any CMS → not applicable.
- Plain PHP without a framework → use `skill-php-8`.

## Non-negotiable rules for Cloude code

1. `declare(strict_types=1);` on the first line of **every** PHP file in the project.
2. **One class per file.** The project namespace is usually `App\` mapped to `app/classes/`.
3. **Mandatory type hints** on every parameter, return and property. No `mixed` unless genuinely polymorphic (JSON decoders, etc.).
4. **Don't invent wrappers** around framework classes. If `Http\Response::json()` does what you need, use it — don't roll your own `header() + echo json_encode()`.
5. **Validate at the edge**: in MCP, `Mcp\Server` validates via `inputSchema`; in HTTP routes, call `JsonSchema::validate($input, $schema)` as early as possible and respond 422 on failure.
6. **No DI container**. Instance state only where it's fundamental (`Logger`, `TaskRunner`, `Mcp\Server`, `AssetUrl` after `configure()`, `Markdown::useParser()`). Everything else is static.
7. **Composer PSR-4**. Don't write custom autoloaders. Don't use `class_alias()` to bridge legacy global names — migrate the call sites with `use` and move on.
8. **Coding style**: `composer cs-check` must pass (PSR-12 / PER-CS 2.0). Use snake_case only in JSON arrays; properties and methods are `camelCase`, classes are `PascalCase`.

## Canonical project layout

```
my-app/
├── www/
│   ├── index.php          ← front controller (template below)
│   └── .htaccess          ← Apache rewrite rules
├── app/
│   ├── config.php         ← defines BASE_URL, DEBUG, DATA_DIR
│   ├── routes.php         ← (optional) router registrations
│   ├── classes/           ← App\ namespace, PSR-4
│   ├── cli/               ← command-line scripts
│   └── views/             ← plain PHP templates
├── data/                  ← JSON / Markdown per entity
├── tests/                    ← Cloude\Testing — run with `vendor/bin/cloude-test`
└── composer.json
```

## Canonical front controller (`www/index.php`)

Memorize it. Do NOT reinvent it. Do NOT add `ob_start()` or `ErrorHandler::register()` by hand — `Bootstrap::run()` already does that.

```php
<?php
declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';
require __DIR__ . '/../app/config.php';   // defines BASE_URL, DEBUG, DATA_DIR

// PHP built-in server static-file passthrough
if (\Cloude\Bootstrap::serveStaticIfExists(__DIR__)) {
    return false;
}

\Cloude\Bootstrap::run(
    debug:    DEBUG,
    viewBase: __DIR__ . '/../app/views',
);

$router = new \Cloude\Router(BASE_URL);
require __DIR__ . '/../app/routes.php';  // or register routes inline
$router->dispatch();
```

## Canonical `app/config.php`

```php
<?php
declare(strict_types=1);

\Cloude\Config::defineBaseUrl([
    'www.example.com',
    'example.com',
    'localhost',
]);
\Cloude\Config::defineDebug();  // defines DEBUG from ENV `DEBUG`

if (!defined('DATA_DIR')) {
    define('DATA_DIR', dirname(__DIR__) . '/data');
}
```

> `defineBaseUrl()` validates `HTTP_HOST` against an allowlist to prevent host-header injection. Non-allowed hosts fall back to `localhost`. **Don't skip this** and don't list hosts dynamically.

## Decision matrix — which class for which task

| You want to… | Use | Notes |
|---|---|---|
| Send a JSON response | `Http\Response::json($data, $status, $pretty)` | NEVER `header()` + `echo json_encode()` by hand |
| 404 / redirect / 204 | `Response::notFound()`, `redirect()`, `noContent()` | `redirect()` already strips CRLF to prevent injection |
| HTML / XML / Markdown / empty 200 | `Response::html/xml/markdown` | |
| Cache a 200 at the CDN | `Http\Cache::ok($seconds)` | Sets both `Cache-Control` and `CDN-Cache-Control` |
| Conditional GET (304) | `Cache::conditionalGet(filemtime($path))` | `true` = already sent 304, exit the route |
| Read JSON (per-request cached) | `JsonFile::read($path)` or `readOr($path, [])` | `null` on missing/invalid |
| Atomic JSON write | `JsonFile::write($path, $data, pretty: false)` | Temp + rename |
| Generic encode/decode | `Format::json/yaml/xml/markdown` | Dispatches `string ↔ array` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polmartinez/cloude-php-workspace](https://github.com/polmartinez/cloude-php-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
