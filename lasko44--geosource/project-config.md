---
trigger: always_on
description: This document defines the architectural decisions and coding standards for this project. All code must follow these guidelines.
---

# Project Coding Standards

This document defines the architectural decisions and coding standards for this project. All code must follow these guidelines.

---

# Architectural Decision Records (ADR)

## Core Architecture

### ADR-001: Skinny Controllers, Fat Services

### Decision
Controllers contain **zero business logic**. All logic lives in service classes.


### Rationale
1. **Testability**: Services can be unit tested without HTTP layer. Controllers need feature tests.
2. **Reusability**: Services can be called from controllers, jobs, commands, or other services.
3. **Readability**: A 50-line controller is easier to understand than a 500-line one.
4. **Single Responsibility**: Each class has one reason to change.
5. **Debugging**: When something breaks, you know where to look based on the type of issue.

### Rules

**Controller responsibilities:**
- Receive HTTP request
- Call service method(s)
- Return HTTP response

**Service responsibilities:**
- Business logic
- Database operations
- External API calls
- Complex calculations

### Example
```php
// ❌ WRONG - Logic in controller
public function store(Request $request) {
    // 50+ lines of validation, authorization, business logic...
}

// ✅ CORRECT - Delegate to service
public function store(StoreScanRequest $request, ScanService $service): RedirectResponse
{
    $scan = $service->executeScan($request->user(), $request->validated());
    return redirect()->route('scans.show', $scan);
}
```

---

### ADR-002: Form Requests for Validation

### Decision
All validation logic lives in **Form Request** classes, not controllers.

### Rationale
1. **Reusability**: Same validation can be used by multiple endpoints.
2. **Separation of concerns**: Controllers don't need to know validation rules.
3. **Authorization co-location**: `authorize()` method keeps auth logic with related validation.
4. **Cleaner controllers**: No `$request->validate([...])` blocks.
5. **Custom error messages**: Centralized in one place.
6. **Data transformation**: Helper methods like `getTier()` encapsulate input transformation.

### Rules
```php
class StoreScanRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Scan::class);
    }

    public function rules(): array
    {
        return [
            'url' => 'required|url',
            'tier' => 'nullable|in:basic,pro,full',
        ];
    }

    public function getTier(): string
    {
        return $this->input('tier', 'basic');
    }
}
```

---

### ADR-003: Policy Classes for Authorization

### Decision
Use **Policy classes** for all model-based authorization, not inline checks.


### Rationale
1. **DRY**: Authorization logic written once, used everywhere.
2. **Testability**: Policies are easily unit tested.
3.**Consistency**: All authorization follows the same pattern.
4.**Discoverability**: All rules for a model are in one file.

### Rules
```php
// Controller - use $this->authorize()
public function show(Scan $scan): Response
{
    $this->authorize('view', $scan);
    // ...
}

// Policy - all authorization logic here
class ScanPolicy
{
    public function view(User $user, Scan $scan): bool
    {
        return $scan->user_id === $user->id
            || $user->allTeams()->contains('id', $scan->team_id);
    }
}
```

---

### ADR-004: Eloquent ORM Only

### Decision
All database queries must use **Eloquent ORM**. Raw `DB::` statements are prohibited.

### Rationale
1. **Consistency**: One way to access the database. No mixing of patterns.
2. **Security**: Eloquent handles parameter binding automatically, reducing SQL injection risk.
3. **Maintainability**: Eloquent queries are more readable and self-documenting.
4. **Model features**: Mass assignment protection, events, observers, and accessors/mutators only work with Eloquent.
5. **Relationships**: Eager loading, lazy loading, and relationship methods require Eloquent.
6. **Testability**: Eloquent models can be easily mocked and factories used for testing.
7. **Refactoring**: Changing table structure only requires updating the model, not hunting for raw queries.

### Prohibited
```php
DB::select('...');
DB::insert('...');
DB::statement('...');
DB::table('...');
DB::raw('...');
```

### Allowed
```php
User::where('status', 'active')->get();
Scan::create(['url' => $url]);
$user->scans()->where('status', 'pending')->get();
```

### Exceptions
Raw expressions (`selectRaw`, `whereRaw`, `orderByRaw`) are allowed for database-specific features like pgvector. Must include a comment explaining why.

---

## Security

### ADR-005: Return 404 Instead of 403

### Decision
Return **404 Not Found** instead of **403 Forbidden** when authorization fails. Log the actual 403 internally.

### Context
When a user attempts to access a resource they're not authorized to view (e.g., another user's scan), we need to decide what HTTP status code to return.

### Rationale
Returning 403 Forbidden leaks information and enables **resource enumeration attacks**:
```
GET /scans/12345 → 403  (attacker learns: scan 12345 EXISTS)
GET /scans/99999 → 404  (attacker learns: scan 99999 doesn't exist)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lasko44) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
