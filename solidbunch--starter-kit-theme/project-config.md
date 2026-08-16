---
trigger: always_on
description: A typical WP theme is procedural: functions in `functions.php`, globals, `add_action('init', 'my_function')`.
---

# PHP conventions

## This is not "classic WordPress" PHP — it's PSR-standard OOP

A typical WP theme is procedural: functions in `functions.php`, globals, `add_action('init', 'my_function')`.
This theme is deliberately not that — every verified pattern below is a real, checkable thing in
this codebase, not an aspiration:

| Standard/pattern | Where it's real, verified |
| --- | --- |
| **PSR-4** autoloading | `composer.json` `autoload.psr-4`: `StarterKit\`→`src/`, `StarterKitBlocks\`→`blocks/`, `StarterKitTests\`→`tests/` (dev-autoload) — no `functions.php`-style function dumps outside `src/dev.php` (see `architecture.md`), which is deliberately global/procedural *because* it's a debug-helper library meant to be callable from anywhere without an import |
| **PSR-12** coding style | Enforced by `phpcs.xml` (`rule ref="PSR12"`), CI-checked (`build-and-ci.md`) |
| **PSR-1**, partially | `phpcs.xml` includes it but excludes `PSR1.Files.SideEffects.FoundWithSymbols` — the one deliberate exception, for the `defined('ABSPATH') || exit;` guard every file needs |
| **PSR-3** (`Psr\Log\LoggerInterface`) | `App::run()` takes a `LoggerInterface` out of the container (`src/App.php:9,41`); bound to a real Monolog logger in `config/common/logger.php` — code depends on the *interface*, not on Monolog directly |
| **PSR-11** (`Psr\Container\ContainerInterface` / `NotFoundExceptionInterface`) | `App`'s container is typed as `ContainerInterface`, not the concrete PHP-DI class (`src/App.php:7,23`); `ConfigEntryNotFoundException` *implements* `Psr\Container\NotFoundExceptionInterface` (`src/Exception/ConfigEntryNotFoundException.php:5,10`) — a config-lookup miss is modeled as a real PSR container exception, not a bespoke one |
| **Dependency Injection container** (`php-di/php-di`) | `config/container.php` + `config/dependencies.php` build the container; swappable via the `starter_kit/container` filter (`architecture.md`) — this is inversion of control, not `new SomeClass()` scattered through hooks |
| **Layered/aggregated configuration** (`laminas/laminas-config-aggregator`) | `config/dependencies.php` merges `config/common/*.php` + env-specific providers into one config array (`architecture.md`) — a Composite-style merge instead of one giant config file or scattered constants |
| **Singleton** (textbook form: private-array registry, protected constructor/`__clone`) | `src/AbstractSingleton.php` — `App extends AbstractSingleton` is the one intentional singleton (the app entrypoint); not used as a general-purpose pattern elsewhere |
| **Repository pattern** | `src/Repository/` — `WpPostRepositoryInterface`/`WpUserRepositoryInterface` + `WpPostRepositoryAbstract` base, concrete `NewsRepository`/`ServiceRepository`/etc. — see the dedicated section below |
| **Program to an interface, not an implementation** (SOLID's "D") | `BlockInterface` (blocks), `CLICommandInterface` (WP-CLI commands), `WpPostRepositoryInterface`/`WpUserRepositoryInterface` (repositories) — each concrete family is built behind an interface, even though WordPress itself never requires this |
| **Fail-fast custom exceptions** instead of silent `null`/`false` | `Helper\Config::get()` throws `ConfigEntryNotFoundException` on a missing key (`architecture.md`) rather than returning `null` — a deliberate departure from WordPress's usual "return false and let the caller maybe check" convention |
| **Chain/pipeline of handlers** | `ErrorHandler::register()` builds a `Whoops\Run` and `pushHandler()`s several handlers in sequence (pretty page, AJAX, REST, plain-text, then a logging closure) — each handler decides whether to act; conceptually the same shape as PSR-15 middleware, applied to error handling |
| **Convention-based registration / auto-discovery** instead of a manual registry | `Handlers\Blocks\Init::loadBlocks()` globs `blocks/*`, and *instantiates a class by string convention* (`{namespace}\{FolderName}\Block`) rather than requiring every block to be hand-registered in `Hooks.php` — same idea `CloneTheme`'s search/replace and the Repository/PostTypes per-entity classes lean on: one place decides the shape, every concrete instance just fills it in |

## Naming case — PSR-1, not WordPress's own coding standard

WordPress's official PHP coding standards mandate **snake_case** for everything — function names
(`register_post_type()`), variables (`$post_id`), hook names (`wp_enqueue_scripts`). This theme's
**own** identifiers follow **PSR-1** instead (`phpcs.xml`'s `PSR1` rule, see above — PSR-1 §4.3
mandates camelCase methods and StudlyCaps class names):

| Identifier kind | Convention here | Evidence |
| --- | --- | --- |
| Classes/interfaces | `StudlyCaps` | `NewsRepository`, `BlockAbstract`, `WpPostRepositoryInterface` |
| Methods, functions, local variables, properties | `camelCase` | `registerBlockArgs()`, `getRecentNews()`, `$blockName`, `$metaPrefix`, `$newThemeDirectory` — never `register_block_args()` or `$block_name` |
| Class constants / `define()`-d constants | `UPPER_SNAKE_CASE` | `SK_PREFIX`, `SK_HOOKS_PREFIX` (PSR-1 §4.4) |
| Block/CPT/Handler directory names | `PascalCase` | `blocks/FaqSection/`, `Handlers/PostTypes/TeamMember.php` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solidbunch/starter-kit-theme](https://github.com/solidbunch/starter-kit-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
