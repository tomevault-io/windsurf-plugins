---
trigger: always_on
description: Refactoring conventions for actions, validators and fractals. Each rule shows a
---

# Code conventions

Refactoring conventions for actions, validators and fractals. Each rule shows a
**Before** (what to avoid) and an **After** (the expected style). The goal is the
same everywhere: small single-purpose methods, no inline branching, no hidden
side effects and explicit data.

---

## 1. Factor array parameters into a method

Inline array literals passed as arguments hide intent and make the call hard to
read. Move them to a dedicated `*Data()` method.

**Before**

```php
protected function expense(): void
{
    $this->factory('Expense')
        ->action([
            'name' => $this->row->name,
            'description' => $this->row->description,
            'amount' => $this->row->amount,
            'date_at' => $this->row->date_at,
            'distance' => $this->row->distance,
            'user_id' => $this->row->user_id,
            'vehicle_id' => $this->row->vehicle_id,
            'maintenance_id' => $this->row->id,
        ])
        ->upsertFromMaintenance();
}
```

**After**

```php
protected function expense(): void
{
    $this->factory('Expense')
        ->action($this->expenseData())
        ->upsertFromMaintenance();
}

protected function expenseData(): array
{
    return [
        'name' => $this->row->name,
        'description' => $this->row->description,
        'amount' => $this->row->amount,
        'date_at' => $this->row->date_at,
        'distance' => $this->row->distance,
        'user_id' => $this->row->user_id,
        'vehicle_id' => $this->row->vehicle_id,
        'maintenance_id' => $this->row->id,
    ];
}
```

---

## 2. In actions `$this->data` already holds validated values

The validator guarantees the shape and presence of every field, so inside the
action you must not re-default values with `?? null`, `?? []` or `?? ''`. Trust
the rules and read the keys directly.

**Action factory**

```php
<?php declare(strict_types=1);

namespace App\Domains\Maintenance\Action;

use App\Domains\Core\Action\ActionFactoryAbstract;
use App\Domains\Maintenance\Model\Maintenance as Model;

class ActionFactory extends ActionFactoryAbstract
{
    protected ?Model $row;

    public function create(): Model
    {
        return $this->actionHandleTransaction(Create::class, $this->validate()->create());
    }
}
```

**Validator**

```php
<?php declare(strict_types=1);

namespace App\Domains\Maintenance\Validate;

use App\Domains\Core\Validate\ValidateAbstract;

class Create extends ValidateAbstract
{
    public function rules(): array
    {
        return [
            'name' => ['bail', 'required'],
            'workshop' => ['bail', 'required'],
            'date_at' => ['bail', 'required', 'date_format:Y-m-d'],
            'distance' => ['bail', 'required', 'numeric'],
            'distance_next' => ['bail', 'numeric'],
            'amount' => ['bail', 'numeric'],
            'description' => ['bail'],
            'user_id' => ['bail', 'integer'],
            'vehicle_id' => ['bail', 'required', 'integer'],
        ];
    }
}
```

**Before**

```php
protected function saveItems(): void
{
    foreach (($this->data['maintenance_item_id'] ?? []) as $index => $item_id) {
        $this->saveItem($item_id, $index);
    }
}

protected function saveItem(int $item_id, int $index): void
{
    MaintenanceMaintenanceItemModel::query()->create([
        'quantity' => $this->data['quantity'][$index] ?? 0,
        'amount_gross' => $this->data['amount_gross'][$index] ?? 0,
        'tax_percent' => $this->data['tax_percent'][$index] ?? 0,
        'maintenance_id' => $this->row->id,
        'maintenance_item_id' => $item_id,
        'user_id' => $this->data['user_id'] ?? null,
    ]);
}
```

**After**

```php
protected function saveItems(): void
{
    foreach ($this->data['maintenance_item_id'] as $index => $item_id) {
        $this->saveItem($item_id, $index);
    }
}

protected function saveItem(int $item_id, int $index): void
{
    MaintenanceMaintenanceItemModel::query()->create([
        'quantity' => $this->data['quantity'][$index],
        'amount_gross' => $this->data['amount_gross'][$index],
        'tax_percent' => $this->data['tax_percent'][$index],
        'maintenance_id' => $this->row->id,
        'maintenance_item_id' => $item_id,
        'user_id' => $this->data['user_id'],
    ]);
}
```

> Keep `??` only when the fallback is a real default derived at runtime (for
> example a loop index), not as a missing-key guard for validated fields.

---

## 3. Do not mix orchestration methods with raw processes

`handle()` (and any orchestration method) should only call other methods. Inline
work such as building the response array belongs in its own method.

**Before**

```php
public function handle(): Model
{
    $this->data();
    $this->check();
    $this->save();
    $this->job();
    $this->expense();

    return $this->row;
}
```

This form is already correct when every step is a method call. Avoid inlining
payload construction or queries inside `handle()`:

**Before (bad)**

```php
public function handle(): array
{
    $this->data();
    $this->check();
    $this->save();

    return [
        'id' => $this->row->id,
        'vehicle_id' => $this->row->vehicle_id,
        'amount' => $this->row->amount,
        'files' => count($this->files),
    ];
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eusonlito/GPS-Tracker](https://github.com/eusonlito/GPS-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
