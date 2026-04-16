---
trigger: always_on
description: Guidance for GitHub Copilot when generating code in the SPIN Framework repository.
---

# GitHub Copilot Instructions — SPIN Framework

Guidance for GitHub Copilot when generating code in the SPIN Framework repository.

## Project Overview

**SPIN Framework** (`celarius/spin-framework`) is a lightweight PHP 8+ web framework emphasizing:
- **JSON-based routing** — routes declared in JSON files, never in PHP
- **Configuration as JSON** — environment-specific config files with macro expansion
- **Middleware pipeline** — structured request/response handling
- **PSR compliance** — PSR-3, PSR-7, PSR-11, PSR-16, PSR-17 implementations
- **Dependency injection** — centralized service container
- **Global helpers** — convenient functions for common operations

## Repository Structure

```
src/
  Application.php                  # Request/response orchestrator
  Core/
    Controller.php                 # HTTP handler base class
    Middleware.php                 # Pipeline handler base class
    Config.php
    Route.php
    RouteGroup.php
    CacheManager.php
    ConnectionManager.php
    Logger.php
  Cache/Adapters/                  # Cache implementations (APCu, Redis, File)
  Database/Drivers/Pdo/            # Database drivers (MySQL, PostgreSQL, SQLite, etc.)
  Helpers/                         # Cipher, JWT, UUID, Hash, ArrayToXml
  Factories/Http/                  # PSR-17 HTTP factories
  Exceptions/                      # Framework exception classes
tests/                             # PHPUnit tests (mirrors src/)
doc/                               # Feature documentation
```

## Naming Conventions

### Files and Directories
- **PSR-4 compliance** — namespace must match directory path
  - `Spin\Core\Controller` → `src/Core/Controller.php`
  - `Spin\Cache\Adapters\RedisCacheAdapter` → `src/Cache/Adapters/RedisCacheAdapter.php`
- **Class names** — PascalCase matching file name exactly
- **Interfaces** — suffix with `Interface` (e.g., `CacheInterface`)
- **Abstract classes** — prefix with `Abstract` (e.g., `AbstractCacheAdapter`)
- **Test files** — mirror source structure, suffix with `Test` (e.g., `UserControllerTest.php`)

### Code Identifiers
- **Constants** — UPPER_SNAKE_CASE
  ```php
  public const DEFAULT_TIMEOUT = 30;
  private const MAX_CONNECTIONS = 10;
  ```
- **Class properties** — camelCase with visibility and type declaration
  ```php
  private string $apiKey;
  protected int $maxRetries;
  public float $timeout;
  ```
- **Methods** — camelCase; HTTP handlers use prefix pattern
  ```php
  public function handleGET(array $args): ResponseInterface { }
  public function handlePOST(array $args): ResponseInterface { }
  private function validateInput(array $data): bool { }
  ```
- **Variables** — camelCase
  ```php
  $userId = 123;
  $userData = ['name' => 'John'];
  ```

## Code Style (PSR-12 + Strict Types)

### File Header
Every PHP file must start with:
```php
declare(strict_types=1);
namespace Spin\Core;
```

### Type Declarations
Always use explicit types; avoid `mixed`:
```php
// Good
public function process(int $id, array $data): string { }

// Bad
public function process($id, $data) { }
```

### Class Structure
```php
declare(strict_types=1);
namespace Spin\Core;

use Psr\Http\Message\ResponseInterface;

/**
 * Brief description of class purpose.
 *
 * @author Your Name
 */
class MyClass
{
    /** @var string */
    private string $apiKey;

    public function __construct(string $apiKey)
    {
        $this->apiKey = $apiKey;
    }

    /**
     * Method description.
     *
     * @param int $id
     * @return string
     */
    public function getData(int $id): string
    {
        return 'data';
    }
}
```

### Method Implementation
```php
public function handleGET(array $args): ResponseInterface
{
    $id = $args['id'] ?? null;

    if (!$id) {
        return responseJson(['error' => 'ID required'], 400);
    }

    try {
        $data = app()->get('service')->fetch($id);
        return responseJson($data);
    } catch (\Exception $e) {
        logger()->error('Fetch failed: ' . $e->getMessage());
        return responseJson(['error' => 'Server error'], 500);
    }
}
```

## Common Patterns

### Controllers
Extend `Spin\Core\Controller` and implement HTTP method handlers:

```php
declare(strict_types=1);
namespace App\Controllers;

use Spin\Core\Controller;
use Psr\Http\Message\ResponseInterface;

class UserController extends Controller
{
    public function handleGET(array $args): ResponseInterface
    {
        $userId = $args['id'] ?? null;
        $user = app()->get('models.user')->find($userId);
        return responseJson($user ?? ['error' => 'Not found'], $user ? 200 : 404);
    }

    public function handlePOST(array $args): ResponseInterface
    {
        $data = getRequest()->getParsedBody();
        $user = app()->get('models.user')->create($data);
        return responseJson($user, 201);
    }
}
```

### Middleware
Extend `Spin\Core\Middleware` and implement both methods:

```php
declare(strict_types=1);
namespace App\Middleware;

use Spin\Core\Middleware;

class AuthMiddleware extends Middleware
{
    public function initialize(array $args): bool
    {
        return true;
    }

    public function handle(array $args): bool
    {
        $token = getRequest()->getHeaderLine('Authorization');

        if (!$token) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Celarius) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
