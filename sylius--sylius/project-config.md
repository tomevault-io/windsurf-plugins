---
trigger: always_on
description: Guidelines for AI assistants contributing to Sylius.
---

# AI Contribution Guidelines

Guidelines for AI assistants contributing to Sylius.

## Reference Files

When working on specific areas, check these files for patterns:

### Entities & Models
- Entity pattern: `src/Sylius/Component/Core/Model/Product.php`
- Interface pattern: `src/Sylius/Component/Core/Model/ProductInterface.php`
- Doctrine mapping: `src/Sylius/Bundle/CoreBundle/Resources/config/doctrine/model/`

### API Platform 4.x
- Resource definitions: `src/Sylius/Bundle/ApiBundle/Resources/config/api_platform/resources/`
- Properties/serialization: `src/Sylius/Bundle/ApiBundle/Resources/config/api_platform/properties/`
- Admin resources: `resources/admin/Product.xml`
- Shop resources: `resources/shop/Product.xml`

### Services & Configuration
- Service definitions: `src/Sylius/Bundle/CoreBundle/Resources/config/services.xml`
- Bundle config: `src/Sylius/Bundle/*/Resources/config/`

### Templates & Hooks
- Admin templates: `src/Sylius/Bundle/AdminBundle/templates/`
- Shop templates: `src/Sylius/Bundle/ShopBundle/templates/`
- Twig hooks: check existing hooks in templates for naming patterns

### Tests
- PHPUnit functional: `tests/Functional/`
- PHPUnit API: `tests/Api/`
- Behat contexts: `src/Sylius/Behat/Context/`

### Migrations
- Location: `src/Sylius/Bundle/CoreBundle/Migrations/`
- **ALWAYS create TWO migrations**: one for MySQL, one for PostgreSQL
- MySQL: extend `Sylius\Bundle\CoreBundle\Doctrine\Migrations\AbstractMigration`
- PostgreSQL: extend `Sylius\Bundle\CoreBundle\Doctrine\Migrations\AbstractPostgreSQLMigration`
- When reviewing PRs that touch migrations, verify both versions exist

## General Guidelines

### Project Structure & Design

- Sylius is an e-commerce framework built on **Symfony**
- Bundles and Components can be used independently
- Follow the Sylius Backward Compatibility (BC) policy
- When changing interfaces, always provide BC layer

### Compatibility & Security

- Ensure compatibility with **Symfony** and **PHP** versions defined in `composer.json`
- For API configuration, use **API Platform 4.x**
- Follow secure coding practices to prevent XSS, CSRF, injections, auth bypasses, etc.

### Coding Standards & Tooling

- Use **4 spaces** for indentation in all files (PHP, YAML, XML, Twig, etc.)
- Use **PHPUnit** for unit and functional testing
- Use **Behat** for behavior-driven scenarios
- Use **ECS** to ensure consistent code style
- Use **PHPStan** for static analysis
- Use **CI** to run all tests and checks automatically

## Commands

- Run `composer install` to install PHP dependencies
- Run `vendor/bin/ecs` to fix PHP code style issues
- Run `vendor/bin/phpstan analyse` to perform static analysis
- Run `vendor/bin/phpunit` to execute unit and functional tests
- Run `yarn install` to install JavaScript dependencies
- Run `yarn encore dev` to compile frontend assets

## PHP Code

- Use modern PHP 8.2+ syntax and features
- Declare `strict_types=1` in all PHP files
- Follow the **Sylius Coding Standard**
- Do not use deprecated features from PHP, Symfony, or Sylius
- Use `final` for all classes, except entities and repositories
- Use `readonly` for immutable services and value objects
- Add type declarations for all properties, arguments, and return values
- Use `camelCase` for variables and method names
- Use `SCREAMING_SNAKE_CASE` for constants
- Use `snake_case` for configuration keys, route names, and template variables
- Use **fast returns** instead of nesting logic unnecessarily
- Use trailing commas in multi-line arrays and argument lists
- Order array keys alphabetically where applicable
- Use PHPDoc only when necessary (e.g. `@var Collection<ProductInterface>`)
- Group class elements in this order: constants, properties, constructor, public methods, protected methods, private methods
- Group getter and setter methods for the same properties together
- Suffix interfaces with Interface, traits with Trait
- Use `use` statements for all non-global classes
- Sort `use` imports alphabetically and group by type (classes, functions, constants)

## Templates and Hooks

- Use modern HTML5 syntax
- Always use the most modern Twig syntax and features
- Icon names must be from the Tabler 1.x library
- Use `snake_case` for all template directory and file names
- Use `snake_case` for all variable names in Twig files
- Ensure the directory structure under `templates/` matches the structure of the corresponding Twig hooks
- Use translations for all strings in templates, never hardcode text

## API

- Define resources in `admin/` and `shop/` folders accordingly
- Define operations in the following order: `get collection`, `get item`, `post`, `put`, `patch`, `delete`
- Define resource serialization in the `serialization/` folder
- Use serialization groups for: `index`, `show`, `create`, `update`
- Use **PHPUnit** tests to validate API configuration and API responses

## PHPUnit

- Place unit tests in `tests/Unit/`
- Place functional tests in `tests/Functional/`
- Place API tests in `tests/Api/`
- Test class names must end with `Test` suffix
- Use `#[Covers]` attribute for unit tests
- For API tests, extend `ApiTestCase` and use `assertResponse*` methods
- Run specific test: `vendor/bin/phpunit --filter TestClassName`

## Behat

- Place feature files in `features/` directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sylius/Sylius](https://github.com/Sylius/Sylius) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
