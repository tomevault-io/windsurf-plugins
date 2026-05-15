---
trigger: always_on
description: This document defines the rules for AI assistants working as **maintainers** of the Fw PHP framework.
---

# Fw Framework - Maintenance & Development Rules

This document defines the rules for AI assistants working as **maintainers** of the Fw PHP framework.

---

## Core Mandate: Framework Evolution

As maintainers, we have full authority to modify the `src/` core, enhance framework features, and refactor internals to improve performance or developer experience.

### Framework Directories (Maintainer Access)

```
src/                    # FRAMEWORK CORE - FULL ACCESS
├── Core/               # Kernel, Routing, Container
├── Database/           # QueryBuilder, Migrations, Connections
├── Model/              # ORM Logic
├── Auth/               # Authentication & Authorization
├── Console/            # CLI Commands & Scaffolding
├── Async/              # Fiber & EventLoop management
└── ...                 # All other framework components
```

### Application & Stubs

```
app/                    # Application code (Models ship; Controllers added by make:spa)
config/                 # Default configuration templates
stubs/                  # Code generation templates (CRITICAL for make:* commands)
database/               # Base migrations and seeders
```

---

## Technical Standards

### 1. PHP 8.4+ Excellence
- Use **Property Hooks** for computed properties in Models/DTOs.
- Use **Asymmetric Visibility** (`public private(set)`) for read-only state.
- Strictly adhere to `declare(strict_types=1);`.

### 2. Async & Concurrency
- Ensure all stateful services are **Fiber-safe**.
- Use `RequestContext` for any data that must not leak between requests in worker mode.
- Prefer non-blocking I/O via the built-in `EventLoop`.

### 3. Error Handling (Monads)
- Return `Result<T, E>` for operations that can fail (Database, API calls).
- Return `Option<T>` for values that might be missing (Find by ID).
- Avoid throwing exceptions for expected control flow.

### 4. Code Generation (Stubs)
- When adding a new scaffolding feature (like `make:spa`), always create high-quality stubs in `stubs/`.
- Ensure stubs are clean, well-commented, and follow the framework's best practices.

---

## Project Structure Convention

Every Fw project MUST follow this exact structure. Do not deviate.

### Required Files

```
project-root/
├── .env                    # Environment variables (from .env.example)
├── .env.example            # Environment template
├── composer.json           # Dependencies
├── fw                      # CLI entry point (executable)
├── public/
│   └── index.php           # Web entry point
├── config/
│   ├── app.php
│   ├── database.php
│   ├── routes.php
│   ├── middleware.php
│   └── providers.php
├── app/
│   ├── Controllers/
│   ├── Models/
│   └── Views/
│       └── layouts/
├── database/
│   └── migrations/
├── storage/
│   ├── cache/
│   ├── logs/
│   └── queue/
└── tests/
```

### Creating New Projects

When starting a new feature or project:

1. **Controllers** go in `app/Controllers/`
   ```php
   // app/Controllers/PostController.php
   namespace App\Controllers;

   use Fw\Core\Controller;
   use Fw\Core\Request;
   use Fw\Core\Response;

   class PostController extends Controller
   {
       public function index(Request $request): Response
       {
           $posts = Post::all();
           return $this->view('posts.index', ['posts' => $posts]);
       }
   }
   ```

2. **Models** go in `app/Models/`
   ```php
   // app/Models/Post.php
   namespace App\Models;

   use Fw\Model\Model;

   class Post extends Model
   {
       protected static ?string $table = 'posts';
       protected static array $fillable = ['title', 'content', 'user_id'];
   }
   ```

3. **Views** go in `app/Views/`
   ```php
   // app/Views/posts/index.php
   <?php $this->layout('main'); ?>

   <?php foreach ($posts as $post): ?>
       <h2><?= $e($post->title) ?></h2>
   <?php endforeach; ?>
   ```

4. **Routes** go in `config/routes.php`
   ```php
   // config/routes.php
   return function (Fw\Core\Router $router): void {
       $router->get('/posts', [App\Controllers\PostController::class, 'index']);
   };
   ```

5. **Migrations** go in `database/migrations/`
   ```bash
   php fw make:migration create_posts_table
   ```

---

## Consistency Requirements

All projects built with Fw MUST follow these patterns. This ensures every project works identically.

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Controllers | PascalCase + `Controller` suffix | `PostController`, `UserController` |
| Models | PascalCase, singular | `Post`, `User`, `Comment` |
| Migrations | snake_case, descriptive | `create_posts_table`, `add_status_to_orders` |
| Views | snake_case, dot notation for folders | `posts.index`, `admin.users.edit` |
| Middleware | PascalCase + `Middleware` suffix | `AuthMiddleware`, `RateLimitMiddleware` |
| Jobs | PascalCase + action name | `SendWelcomeEmail`, `ProcessPayment` |
| Config keys | snake_case | `app.debug`, `database.default` |
| Routes | kebab-case URLs | `/user-profile`, `/blog-posts` |
| Tables | snake_case, plural | `users`, `blog_posts`, `order_items` |

### Controller Patterns

Always follow this structure:

```php
<?php

declare(strict_types=1);

namespace App\Controllers;

use Fw\Core\Controller;
use Fw\Core\Request;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velkymx/vibefw](https://github.com/velkymx/vibefw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
