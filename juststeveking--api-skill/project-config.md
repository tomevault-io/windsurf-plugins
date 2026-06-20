---
trigger: always_on
description: Provides opinionated best practices and patterns for building production-ready REST APIs using Laravel. Use this skill when designing API endpoints, implementing resource controllers, or structuring JSON responses in a Laravel environment.
---


# API Skill for Laravel Developers

This skill defines the exact patterns and rules for building scalable, reliable, and modern REST APIs in Laravel. All guidance here is prescriptive. When in doubt, follow the rule.

---

## 1. Route Organisation

Standalone APIs have **no `api` prefix** on any route. Routes live under `routes/api/` as follows:

```
routes/
  api/
    routes.php       ← entry point, requires all resource files
    auth.php
    posts.php        ← one file per resource
    users.php
```

`routes/api/routes.php` loads in each resource using a prefix and naming:

```php
<?php

declare(strict_types=1);

use Illuminate\Support\Facades\Route;

Route::as('auth:')->group(base_path(
    path: 'routes/api/auth.php',
));

Route::as('posts:')->group(base_path(
    path: 'routes/api/posts.php',
));
```

Each resource file owns its own version prefix. This keeps versioning explicit and debuggable per resource:

```php
<?php

declare(strict_types=1);

use App\Http\Controllers\Posts;
use Illuminate\Support\Facades\Route;

Route::prefix('v1/posts')->middleware(['auth:sanctum', 'throttle:api'])->group(function (): void {
    Route::get('/', Posts\V1\IndexController::class)->name('v1:index');
    Route::post('/', Posts\V1\StoreController::class)->name('v1:store');
    Route::get('/{post}', Posts\V1\ShowController::class)->name('v1:show');
    Route::put('/{post}', Posts\V1\UpdateController::class)->name('v1:update');
    Route::delete('/{post}', Posts\V1\DestroyController::class)->name('v1:destroy');
});
```

- Always version from day one.
- Always use named routes, namespaced to their version (e.g. `posts:v1:index` and `posts:v1:store`).
- The `throttle:api` middleware must always be present.

---

## 2. Single-Action Invokable Controllers

Every controller is a `final` single-action invokable class. No resourceful controllers. No multiple methods per class. Just an invokable action and a constructor for any dependency injection.

Controllers live under `app/Http/Controllers/{Resource}/{Version}/`:

```
app/Http/Controllers/Posts/V1/
  IndexController.php
  ShowController.php
  StoreController.php
  UpdateController.php
  DestroyController.php
```

Dependencies are always injected via the constructor. Never use Facades, `app()`, or `resolve()` inside a controller. The `__invoke` method handles the request and returns a response:

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Posts\V1;

use App\Actions\Posts\StorePostAction;
use App\Http\Requests\Posts\V1\StoreRequest;
use App\Http\Resources\PostResource;
use Illuminate\Http\JsonResponse;
use Symfony\Component\HttpFoundation\Response;

final class StoreController
{
    public function __construct(
        private readonly StorePostAction $action,
    ) {}

    public function __invoke(StoreRequest $request): JsonResponse
    {
        $post = $this->action->handle(
            payload: $request->payload(),
        );

        return new JsonResponse(
            data: new PostResource($post),
            status: Response::HTTP_CREATED,
        );
    }
}
```

---

## 3. Form Requests and Payloads (DTOs)

Every state-mutating endpoint uses a **Form Request**. Form Requests live under `app/Http/Requests/{Resource}/{Version}/`.

The Form Request **must** expose a `payload()` method that returns a typed DTO from `app/Http/Payloads/`. This keeps controllers free of array handling and makes the data contract explicit:

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests\Posts\V1;

use App\Http\Payloads\Posts\StorePayload;
use Illuminate\Foundation\Http\FormRequest;

final class StoreRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title'   => ['required', 'string', 'max:255'],
            'content' => ['required', 'string'],
        ];
    }

    public function payload(): StorePayload
    {
        return new StorePayload(
            title:   $this->string('title')->toString(),
            content: $this->string('content')->toString(),
            userId:  $this->user()->id,
        );
    }
}
```

**Payloads (DTOs)** are plain PHP objects. They have a typed constructor and a `toArray()` method that returns what Eloquent expects. They live in `app/Http/Payloads/`:

```php
<?php

declare(strict_types=1);

namespace App\Http\Payloads\Posts;

final class StorePayload
{
    public function __construct(
        public readonly string $title,
        public readonly string $content,
        public readonly string $userId,
    ) {}

    public function toArray(): array
    {
        return [
            'title'   => $this->title,
            'content' => $this->content,
            'user_id' => $this->userId,
        ];
    }
}
```

---

## 4. API Resources

Always use Laravel's Eloquent API Resources to transform model data. Generate them with the `--json-api` CLI flag:

```bash
php artisan make:resource PostResource --json-api
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustSteveKing/api-skill](https://github.com/JustSteveKing/api-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
