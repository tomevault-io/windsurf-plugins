---
trigger: always_on
description: > This document helps AI coding agents (Claude, Copilot, Cursor, etc.) understand the MonkeysLegion framework conventions and architecture. Place this file at the project root as `AGENTS.md`.
---

# MonkeysLegion v2 — AI Agent Guide

> This document helps AI coding agents (Claude, Copilot, Cursor, etc.) understand the MonkeysLegion framework conventions and architecture. Place this file at the project root as `AGENTS.md`.

## Quick Facts

- **Language:** PHP 8.4+ (uses property hooks, `readonly`, enums, attributes)
- **Architecture:** PSR-7/PSR-15 HTTP, PSR-11 DI container, attribute-based routing
- **Config format:** `.mlc` files (MonkeysLegion Config — custom HOCON-like format)
- **Template engine:** `.ml.php` files (Blade-like syntax)
- **Entry point:** `public/index.php` → `Application::create(basePath: ...)->run()`
- **Namespace:** App code lives under `App\` (PSR-4 mapped to `app/`)
- **Tests:** PHPUnit 11+ in `tests/` (Unit, Integration, Feature, Performance)

---

## Project Structure

```
├── app/                    # Application code (App\ namespace)
│   ├── Controller/         # HTTP controllers (auto-discovered)
│   │   └── Api/            # API controllers (usually prefixed)
│   ├── Dto/                # Request DTOs with validation attributes
│   ├── Entity/             # Database entities with attribute mappings
│   ├── Enum/               # PHP 8.1+ enums
│   ├── Event/              # Domain events
│   ├── Job/                # Queue jobs
│   ├── Listener/           # Event listeners
│   ├── Middleware/          # Custom PSR-15 middleware
│   ├── Policy/             # Authorization policies
│   ├── Providers/          # Service providers (DI definitions)
│   ├── Repository/         # Data access (extends EntityRepository)
│   ├── Resource/           # API response transformers
│   └── Service/            # Business logic layer
├── config/                 # Configuration
│   ├── app.mlc             # Core app settings
│   ├── app.php             # DI container overrides (interface bindings)
│   ├── database.mlc        # Database connections
│   ├── middleware.mlc       # Global middleware pipeline
│   └── *.mlc               # Other config (auth, cache, cors, etc.)
├── resources/views/        # Templates (.ml.php files)
│   └── layouts/            # Layout templates
├── public/                 # Web root
│   ├── index.php           # Front controller (DO NOT MODIFY)
│   └── assets/             # Static files (CSS, JS, images)
├── database/migrations/    # SQL migration files
├── tests/                  # PHPUnit test suites
├── server.php              # PHP built-in server router
└── bin/                    # CLI scripts
```

---

## Creating a Controller

Controllers live in `app/Controller/` and are auto-discovered. **No registration needed.**

### Web Controller (returns HTML)
```php
<?php
declare(strict_types=1);

namespace App\Controller;

use MonkeysLegion\Router\Attributes\Route;
use MonkeysLegion\Http\Message\Response;
use MonkeysLegion\Template\Renderer;

final class ProductController
{
    public function __construct(
        private readonly Renderer $renderer,
    ) {}

    #[Route(methods: 'GET', path: '/products', name: 'products.index')]
    public function index(): Response
    {
        return Response::html($this->renderer->render('products.index', [
            'title' => 'Products',
            'products' => [], // pass data to template
        ]));
    }

    #[Route(methods: 'GET', path: '/products/{id:\d+}', name: 'products.show')]
    public function show(ServerRequestInterface $request, string $id): Response
    {
        // $id comes from the URL parameter
        return Response::html($this->renderer->render('products.show', [
            'product' => $this->repo->findOrFail((int) $id),
        ]));
    }
}
```

### API Controller (returns JSON)
```php
<?php
declare(strict_types=1);

namespace App\Controller\Api;

use MonkeysLegion\Router\Attributes\Route;
use MonkeysLegion\Router\Attributes\RoutePrefix;
use MonkeysLegion\Router\Attributes\Middleware;
use MonkeysLegion\Auth\Attribute\Authenticated;
use MonkeysLegion\Http\Message\Response;
use Psr\Http\Message\ServerRequestInterface;

#[RoutePrefix('/api/v2/products')]
#[Middleware(['cors'])]
final class ProductApiController
{
    public function __construct(
        private readonly ProductService $service,
        private readonly ProductRepository $products,
    ) {}

    #[Route('GET', '/', name: 'api.products.index', summary: 'List products', tags: ['Products'])]
    public function index(): Response
    {
        return Response::json(['data' => $this->products->findAll()]);
    }

    #[Route('POST', '/', name: 'api.products.create', summary: 'Create product', tags: ['Products'])]
    #[Authenticated]
    public function create(CreateProductRequest $dto): Response
    {
        $product = $this->service->create($dto);
        return Response::json(['data' => $product], 201);
    }
}
```

### Route Attribute Options
```php
#[Route(
    methods: 'GET',              // HTTP method(s): 'GET', 'POST', ['GET', 'POST']
    path: '/users/{id:\d+}',    // Path with regex constraints
    name: 'users.show',         // Named route for URL generation
    summary: 'Get user',        // OpenAPI summary
    tags: ['Users'],            // OpenAPI tags

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MonkeysCloud/MonkeysLegion-Skeleton](https://github.com/MonkeysCloud/MonkeysLegion-Skeleton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
