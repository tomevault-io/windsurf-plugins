---
trigger: always_on
description: This document covers folder structure, naming conventions, and complete worked examples for the API skill.
---

# Conventions Reference

This document covers folder structure, naming conventions, and complete worked examples for the API skill.

---

## Folder Structure

```
app/
  Actions/
    Posts/
      StorePostAction.php
      UpdatePostAction.php
      DestroyPostAction.php
  Http/
    Controllers/
      Auth/
        V1/
          LoginController.php
          LogoutController.php
          RegisterController.php
      Posts/
        V1/
          IndexController.php
          ShowController.php
          StoreController.php
          UpdateController.php
          DestroyController.php
    Middleware/
      ForceJsonResponse.php
      Sunset.php
    Payloads/
      Posts/
        StorePayload.php
        UpdatePayload.php
      Auth/
        RegisterUserPayload.php
    Requests/
      Auth/
        V1/
          LoginRequest.php
          RegisterRequest.php
      Posts/
        V1/
          StoreRequest.php
          UpdateRequest.php
    Resources/
      PostResource.php
      UserResource.php
    Responses/
      ProblemResponse.php
  Jobs/
    Posts/
      StorePostJob.php
  Policies/
    PostPolicy.php
routes/
  api/
    routes.php
    auth.php
    posts.php
tests/
  Feature/
    Auth/
      V1/
        LoginTest.php
        RegisterTest.php
    Posts/
      V1/
        IndexTest.php
        ShowTest.php
        StoreTest.php
        UpdateTest.php
        DestroyTest.php
```

---

## Naming Conventions

| Layer | Convention | Example |
|---|---|---|
| Controller | `{Action}Controller` | `StoreController`, `DestroyController` |
| Action | `{Action}{Resource}Action` | `StorePostAction`, `UpdatePostAction` |
| Payload (DTO) | `{Action}Payload` | `StorePayload` |
| Form Request | `{Action}Request` | `StoreRequest` |
| API Resource | `{Resource}Resource` | `PostResource` |
| Job | `{Action}{Resource}Job` | `StorePostJob` |
| Route name | `{resource}:{version}:{action}` | `posts:v1:store` |
| Test file | `{Action}Test` in the matching path | `StoreTest.php` |

---

## Route Files

### `routes/api/routes.php`

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

### `routes/api/auth.php`

```php
<?php

declare(strict_types=1);

use App\Http\Controllers\Auth;
use Illuminate\Support\Facades\Route;

Route::prefix('v1/auth')->middleware('throttle:api')->group(function (): void {
    Route::post('/register', Auth\V1\RegisterController::class)->name('v1:register');
    Route::post('/login', Auth\V1\LoginController::class)->name('v1:login');

    Route::middleware('auth:sanctum')->group(function (): void {
        Route::delete('/logout', Auth\V1\LogoutController::class)->name('v1:logout');
    });
});
```

### `routes/api/posts.php`

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

---

## Model — ULID Primary Keys

All API-facing models use `HasUlids`. The migration column must be `ulid`:

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

final class Post extends Model
{
    use HasUlids;

    protected function casts(): array
    {
        return [
            'published_at' => 'datetime',
        ];
    }
}
```

Migration:

```php
$table->ulid('id')->primary();
```

Never use `$table->id()` (auto-increment) on a model that is exposed through an API endpoint.

---

## Complete Worked Example — Storing a Post

### Payload

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

### Form Request

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustSteveKing/api-skill](https://github.com/JustSteveKing/api-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
