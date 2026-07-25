---
trigger: always_on
description: FOSSBilling is a free and open-source billing and client management solution designed for hosting businesses and other online service providers. It automates invoicing, payment processing, and client management while being extensible and easily integrable with server management software and payment gateways. The project is primarily written in PHP with modern frontend technologies.
---

# FOSSBilling Project

## Project Overview

FOSSBilling is a free and open-source billing and client management solution designed for hosting businesses and other online service providers. It automates invoicing, payment processing, and client management while being extensible and easily integrable with server management software and payment gateways. The project is primarily written in PHP with modern frontend technologies.

### Key Technologies

* **Backend:** PHP 8.3+ with dependencies managed by Composer. Key libraries include:
  * [Symfony Components](https://symfony.com/): Console, cache, filesystem, HTTP client, and other core functionalities. See `composer.json` for a list of imported components.
    * Prefer Symfony components wherever you can.
    * Use `Filesystem`, `Path`, and `Finder` for filesystem operations instead of native PHP functions (e.g., `$filesystem->exists()` instead of `file_exists()`, `Finder` instead of `glob()`).
  * [Twig](https://twig.symfony.com/): Template engine for rendering views
    * API endpoints are injected as parameters to Twig. See the "Interacting with the FOSSBilling API" section.
    * Twig environments are created via `TwigFactory` at `src/library/FOSSBilling/Twig/TwigFactory.php`.
    * Three environment types: **admin**, **client**, and **email** (sandboxed for security).
    * Email templates use a sandboxed environment (`EmailPolicy.php`) that restricts allowed tags/filters/functions.
  * [RedBeanPHP](https://redbeanphp.com/): ORM for database interactions in legacy modules.
  * [Doctrine DBAL/ORM](https://doctrine-project.org/): ORM and DBAL for modern modules.
    * FOSSBilling is in the process of migrating modules and core parts from RedBeanPHP to Doctrine one by one.
    * The entity manager is available as `$di['em']`. It comes from the EntityManagerFactory in `/src/library/FOSSBilling/Doctrine/EntityManagerFactory.php`.
    * Entities and repositories reside under `/src/modules/*/Entity/{Entity}.php` and `/src/modules/*/Repository/{EntityRepository}.php`.
    * The FOSSBilling project is in the process of gradually phasing out RedBeanPHP in favor of Doctrine ORM.
    * When writing new pieces of code, avoid RedBeanPHP.
    * If you are assisting with the migration from RedBeanPHP to Doctrine, do your best to keep compatibility with the existing table structure.
    * When refactoring API endpoints, check how the `$di['pager']` works in `src/library/FOSSBilling/Pagination.php`. `paginateDoctrineQuery()` is the replacement for `getPaginatedResultSet()`.
  * [Monolog](https://github.com/Seldaek/monolog): Logging framework. Used via `$di['logger']` (`/src/library/FOSSBilling/Monolog.php`).
  * [dompdf](https://github.com/dompdf/dompdf): PDF generation for invoices and documents
  * [Pimple](https://github.com/silexphp/Pimple): Dependency injection container, see `src/di.php`.
* **Frontend:** Modern JavaScript and CSS with npm package management. Key dependencies include:
  * [Tabler.io](https://tabler.io): CSS framework for the admin theme, based on [Bootstrap 5](https://getbootstrap.com/)
  * [Bootstrap 5](https://getbootstrap.com/): CSS framework used directly by the Huraga client theme
  * [CKEditor 5](https://ckeditor.com/ckeditor-5/): Shared rich text editor built into the core public assets
  * [Tom Select](https://tom-select.js.org/): Enhanced select boxes with search and tagging
  * [Autosize](http://www.jacklmoore.com/autosize/): Automatic textarea resizing
  * [Flag Icons](https://flagicons.lipis.dev/): Country flag icon library
  * Use vanilla JavaScript for all JS code.
* **Build Tools:**
  * [esbuild](https://esbuild.github.io/): Fast JavaScript/CSS bundler and minifier
  * [Sass](https://sass-lang.com/): CSS preprocessing
  * [PostCSS](https://postcss.org/) with Autoprefixer: CSS post-processing
  * [svg-sprite](https://github.com/svg-sprite/svg-sprite): SVG sprite generation for icons
* **Testing:**
  * [Pest](https://pestphp.com/): Unit, module, and live API test framework
* **Code Quality & Analysis:**
  * [PHP-CS-Fixer](https://cs.symfony.com/): PSR-12 coding standards enforcement
  * [Rector](https://getrector.org/): Automated PHP code refactoring and modernization
  * [PHPStan](https://phpstan.org/): Static analysis for PHP code

### Architecture

FOSSBilling follows a modular architecture with clear separation of concerns:

* **Core Application:** Located in `src/` directory containing the main application logic
* **Modules:** Located in `src/modules/` - Two types of modules exist:
  * **Service Modules:** Represent products that can be sold (e.g., hosting packages, downloadable products). These modules' names must start with "Service", such as "Servicehosting".
  * **Extension Modules:** Extend FOSSBilling with additional functionality
  * Module templates are organized in `templates/admin/`, `templates/client/`, and `templates/email/` subdirectories.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FOSSBilling/FOSSBilling](https://github.com/FOSSBilling/FOSSBilling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
