---
trigger: always_on
description: Development rules to follow while developing the project
---


# Laravel Development Standards for Cursor AI

> **Note:** This is a general Laravel development guide. For project-specific requirements, patterns, and configurations, see `laravel-blog-api-rules.mdc`.

**Tech Stack:** PHP 8.2+, Laravel 12+, MySQL 8+

## Core Principles

- **SOLID**, **YAGNI**, **KISS**
- **Strict typing**: `declare(strict_types=1);` in all PHP files
- **Thin controllers**: No business logic
- **Type safety**: All properties, parameters, return types explicit
- **PHPStan level 10**: No ignored errors (project requirement)

## PHP Standards

### Type Safety (Mandatory)
- Scalar type hints, return types, property types
- Union types (`string|int`), nullable types (`?Type`)
- Readonly properties for DTOs
- Enums for fixed value sets

```php
declare(strict_types=1);

enum UserStatus: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

class UserDTO
{
    public function __construct(
        public readonly string $email,
        public readonly UserStatus $status
    ) {}
}
```

### Modern PHP Features
- Readonly properties, Enums, Match expressions
- Named arguments, Attributes (PHP 8.0+)
- Constructor property promotion

## Laravel Architecture

### Project Structure
```
app/
├── Actions/              # Complex orchestration (use sparingly per YAGNI)
├── Data/                 # DTOs (create as needed)
├── Enums/                # Type-safe constants
├── Http/
│   ├── Controllers/      # Thin, API/V1/
│   ├── Requests/         # Form Requests
│   └── Resources/        # API Resources
├── Models/
├── Policies/             # Authorization
├── Repositories/         # Data access layer (use per decision matrix)
│   ├── Contracts/         # Interfaces
│   └── Eloquent/          # Implementations
└── Services/             # Business logic
```

Note: This is a general structure. See project-specific rules for exact directory structure used in this project.

### Controllers (Thin)
- Only: validation, authorization, delegation, response formatting
- Use Form Requests, Policies, Services/Actions
- Return typed `JsonResponse` with Resources
- Use dependency injection with readonly properties
- Be final classes for immutability
- Note: For this project, use invokable pattern with `__invoke()` method (see project-specific rules)

### Form Requests
- Always use for validation
- Implement `authorize()` when needed
- Custom messages and attributes
- Note: For this project, implement `withDefaults()` method (see project-specific rules)

### API Resources
- Always use for API responses
- Separate resources for different contexts
- Note: For this project, use specific response format with `status`, `message`, and `data` fields (see project-specific rules)

## Pattern Stack (YAGNI)

### Decision Matrix
| Complexity | Action | Service | Repository | DTO |
|------------|--------|---------|------------|-----|
| Complex (multi-service) | ✅ | ✅ | ✅ | ✅ |
| Medium (business logic) | ❌ | ✅ | ✅ | ✅ |
| Simple (data access) | ❌ | ❌ | ✅ | ✅ |
| Very Simple | ❌ | ❌ | ✅ | ⚠️ |

### Complex Operations
```
Controller → Action → Service → Repository → Model
            ↓
          DTO
```

### Simple CRUD
```
Controller → Service → Repository → Model
            ↓
          DTO
```

### DTOs (Always Recommended)
- Prevent array abuse, ensure type safety
- Readonly properties, `toArray()` method
- Static factory methods

```php
final class UserDTO
{
    public function __construct(
        public readonly string $email,
        public readonly string $password,
        public readonly ?string $name = null,
    ) {}

    public static function fromRequest(StoreUserRequest $request): self
    {
        return new self(
            email: $request->validated('email'),
            password: $request->validated('password'),
            name: $request->validated('name'),
        );
    }

    public function toArray(): array
    {
        return [
            'email' => $this->email,
            'password' => $this->password,
            'name' => $this->name,
        ];
    }
}
```

### Services
- Business logic only
- Depend on repositories, not models
- Constructor injection

```php
final class UserService
{
    public function __construct(
        private readonly UserRepositoryInterface $repository
    ) {}

    public function register(UserDTO $dto): User
    {
        return $this->repository->create($dto->toArray());
    }
}
```

### Repositories
- Data access only
- Interface + Eloquent implementation
- Bind in service provider

```php
interface UserRepositoryInterface
{
    public function create(array $data): User;
    public function findById(int $id): ?User;
}

class EloquentUserRepository implements UserRepositoryInterface
{
    public function create(array $data): User
    {
        return User::create($data);
    }
}
```

### Actions (Complex Orchestration Only)
- Single-purpose operations
- Orchestrate multiple services
- Use only when needed (YAGNI)

```php
final class RegisterUserAction
{
    public function __construct(
        private readonly UserService $userService,
        private readonly EmailService $emailService
    ) {}

    public function execute(UserDTO $dto): User
    {
        $user = $this->userService->register($dto);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mubbi/laravel-blog-api](https://github.com/mubbi/laravel-blog-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
