---
trigger: always_on
description: This document provides guidelines and best practices for structuring and developing a project using the FlightPHP framework.
---

# FlightPHP Skeleton Project Instructions

This document provides guidelines and best practices for structuring and developing a project using the FlightPHP framework.

## Instructions for AI Coding Assistants

As you are developing this project, follow these guidelines as close as you can. If you are unsure about something, ask for clarification before proceeding. You should feel 95% confident in the coding decisions that you make, but allow yourself an offramp if you are not sure about something to ask questions.

## Project Structure

Organize your project as follows:

project-root/
│
├── app/                # Application-specific code
│   ├── commands/       # Custom CLI commands for Runway (built using adhocore/cli)
│   ├── config/         # Configuration files (database, app settings, routes)
│                       # Key files in this folder:
│                       #   - bootstrap.php: Bootstraps and connects the files in the config folder.
│                       #   - routes.php: Route definitions.
│                       #   - services.php: Service definitions (where config variables are used and injected).
│   ├── controllers/    # Route controllers (e.g., HomeController.php)
│   ├── logic/          # (For large projects) Business logic classes/services, called from controllers
│   ├── middlewares/    # Custom middleware classes/functions
│   ├── models/         # Data models (if needed, usually using flightphp/active-record)
│   ├── utils/          # Utility/helper functions
│   └── views/          # View templates (if using)
│
├── public/             # Web root (index.php, assets, etc.)
│
├── vendor/             # Composer dependencies
│
├── tests/              # Unit and integration tests
│
├── composer.json       # Composer config
│
└── README.md           # Project overview

## Development Guidelines

- **Controllers:** Place all route-handling logic in `app/controllers/`. Each controller should handle a specific resource or feature. For large projects, move business logic out of controllers and into the `app/logic/` directory as dedicated classes/services, and call them from your controllers. Use appropriate namespaces for organization. By default, all controllers inject the `Engine $app` variable unless this project has its own dependency injection handler.
- **Namespaces:** Use lowercase namespaces for all classes in the `app/` directory. For example, `app/controllers/HomeController.php` should have the namespace `app\controllers`.
- **Middlewares:** Store reusable middleware in `app/middlewares/`. Register them in your bootstrap or route files.
- **Utils:** Place helper functions and utilities in `app/utils/`.
- **Models:** If your app uses data models, keep them in `app/models/`.
- **Views:** Store templates in `app/views/` if using a templating engine.
- **Config:** Use the `app/config/` directory for configuration files. The main config file is `config.php`, which should be created by copying `config_sample.php` and updating as needed. In other main bootstrap files like bootstrap, and services.php, the $config variable is available to use to access configuration values.
- **Public:** Only expose the `public/` directory to the web server. All requests should go through `public/index.php`.
- **Environment:** Do not use .env files; all configuration should be managed in `app/config/config.php`.
- **Routes:** Define routes in `app/config/routes.php`. Use the `$router->map()` method to register routes with all request methods or `$router->get()` for `GET $router->post()` for POST etc. and associate them with controller methods. Best practice for defining the controller is [ MyController::class, 'myMethod' ].

### Creating new Controllers

First register the view system in your bootstrap file.

```php
// Configure view system with layout
$app->register('view', \flight\template\View::class, [], function($view) {
    $view->path = __DIR__ . '/../views';
    $view->extension = '.php';
});
```

And in your controller:

```php
// Set the content for the layout
$content = $this->app->view()->fetch('catalogo/index', $data);
$data['content'] = $content;

// Render with layout
$this->app->render('layout', $data);
```

## Getting Started

1. Clone the repository and run `composer install`.
2. Copy `app/config/config_sample.php` to `app/config/config.php` and update configuration values as needed.
3. Set your web server's document root to the `public/` directory.
4. Add new controllers, middlewares, and utilities as needed, following the structure above.
5. Register routes and middlewares in your bootstrap file (usually `public/index.php`).

## CLI Commands

Flight projects can include custom CLI commands to automate tasks such as migrations, seeding, or maintenance. The recommended CLI tool is [flightphp/runway](https://github.com/flightphp/runway), which builds on the [adhocore/cli](https://github.com/adhocore/cli) package (not Symfony Console).

- Place your custom command classes in the `app/commands/` directory.
- Runway will automatically discover and register commands from this directory.
- All CLI commands should be built using the adhocore/cli package (do not use Symfony Console).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dougkusanagi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
