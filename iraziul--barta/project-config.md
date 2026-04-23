---
trigger: always_on
description: These guidelines ensure consistency across the Barta SMS package codebase.
---

# Barta Package Development Guidelines

These guidelines ensure consistency across the Barta SMS package codebase.

## PHP Code Style

### General Rules

- Always use `declare(strict_types=1);` at the top of every PHP file
- Use PHP 8.4+ features (constructor promotion, named arguments, readonly, etc.)
- Follow PSR-12 coding standards (enforced by Laravel Pint)
- Run `composer format` before committing

### Class Structure

```php
<?php

declare(strict_types=1);

namespace Larament\Barta\FeatureName;

// Imports grouped: PHP core, Laravel, Package internal
use Exception;
use Illuminate\Support\Facades\Http;
use Larament\Barta\Data\ResponseData;

final class ClassName
{
    // 1. Properties (with type hints)
    // 2. Constructor
    // 3. Public methods
    // 4. Protected methods
    // 5. Private methods
}
```

### Naming Conventions

| Element     | Convention                   | Example                         |
| ----------- | ---------------------------- | ------------------------------- |
| Classes     | PascalCase, singular         | `EsmsDriver`, `SendSmsJob`      |
| Methods     | camelCase, verb-first        | `send()`, `formatPhoneNumber()` |
| Properties  | camelCase                    | `$recipients`, `$baseUrl`       |
| Config keys | snake_case                   | `api_token`, `sender_id`        |
| Env vars    | UPPER_SNAKE_CASE with prefix | `BARTA_ESMS_TOKEN`              |

### Class Modifiers

- Use `final` for concrete implementations (drivers, jobs)
- Use `abstract` for base classes meant to be extended
- Use `readonly` for immutable data objects
- Combine traits on single line: `use Trait1, Trait2, Trait3;`

---

## Driver Development

### Creating a New Driver

1. Extend `AbstractDriver`
2. Mark as `final class`
3. Implement `send(): ResponseData`
4. Override `validate()` to check driver-specific config
5. Use `$this->recipients` (array) and `$this->message` (string)

### Driver Template

```php
<?php

declare(strict_types=1);

namespace Larament\Barta\Drivers;

use Illuminate\Support\Facades\Http;
use Larament\Barta\Data\ResponseData;
use Larament\Barta\Exceptions\BartaException;

final class NewGatewayDriver extends AbstractDriver
{
    private string $baseUrl = 'https://api.gateway.com';

    public function send(): ResponseData
    {
        $this->validate();

        $response = Http::baseUrl($this->baseUrl)
            ->withToken($this->config['api_token'])
            ->timeout($this->timeout)
            ->retry($this->retry, $this->retryDelay)
            ->acceptJson()
            ->post('/sms/send', [
                'to' => implode(',', $this->recipients),
                'message' => $this->message,
                'sender' => $this->config['sender_id'],
            ])
            ->json();

        if ($response['status'] !== 'success') {
            throw new BartaException($response['error']);
        }

        return new ResponseData(
            success: true,
            data: $response,
        );
    }

    protected function validate(): void
    {
        parent::validate();

        if (! $this->config['api_token']) {
            throw new BartaException('Please set api_token for NewGateway in config/barta.php.');
        }

        if (! $this->config['sender_id']) {
            throw new BartaException('Please set sender_id for NewGateway in config/barta.php.');
        }
    }
}
```

### Driver Checklist

- [ ] Extends `AbstractDriver`
- [ ] Uses `final class`
- [ ] Calls `$this->validate()` first in `send()`
- [ ] Uses `implode(',', $this->recipients)` for bulk support
- [ ] Uses `$this->timeout`, `$this->retry`, `$this->retryDelay`
- [ ] Returns `ResponseData` object
- [ ] Throws `BartaException` on API errors
- [ ] Validates required config in `validate()`
- [ ] Registered in `BartaManager`
- [ ] Config added to `config/barta.php`
- [ ] Has comprehensive tests

---

## Testing Guidelines

### Test File Structure

- Location: `tests/` mirrors `src/` structure
- Naming: `{ClassName}Test.php`
- Use Pest PHP (not PHPUnit classes)

### Test Patterns

```php
<?php

declare(strict_types=1);

use Illuminate\Support\Facades\Http;
use Larament\Barta\Drivers\NewDriver;
use Larament\Barta\Exceptions\BartaException;

beforeEach(function () {
    // Set up config for each test
    config()->set('barta.drivers.new.api_token', 'test_token');
    config()->set('barta.drivers.new.sender_id', 'test_sender');
});

it('can instantiate the driver', function () {
    $driver = new NewDriver(config('barta.drivers.new'));
    expect($driver)->toBeInstanceOf(NewDriver::class);
});

it('sends sms successfully', function () {
    Http::fake([
        'https://api.gateway.com/*' => Http::response(['status' => 'success'], 200),
    ]);

    $driver = new NewDriver(config('barta.drivers.new'));
    $response = $driver->to('8801700000000')->message('Test')->send();

    expect($response->success)->toBeTrue();

    Http::assertSent(function ($request) {
        return str_contains($request->url(), 'gateway.com') &&
               $request['message'] === 'Test';
    });
});

it('throws exception on api error', function () {
    Http::fake([
        '*' => Http::response(['status' => 'error', 'error' => 'Failed'], 200),
    ]);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iRaziul/barta](https://github.com/iRaziul/barta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
