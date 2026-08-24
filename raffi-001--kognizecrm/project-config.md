---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== .ai/v4 rules ===

## Filament 4

### Important Version 4 Changes
- File visibility is now `private` by default.
- The `deferFilters` method from Filament v3 is now the default behavior in Filament v4, so users must click a button before the filters are applied to the table. To disable this behavior, you can use the `deferFilters(false)` method.
- The `Grid`, `Section`, and `Fieldset` layout components no longer span all columns by default.
- The `all` pagination page method is not available for tables by default.
- All action classes extend `Filament\Actions\Action`. No action classes exist in `Filament\Tables\Actions`.
- The `Form` & `Infolist` layout components have been moved to `Filament\Schemas\Components`, for example `Grid`, `Section`, `Fieldset`, `Tabs`, `Wizard`, etc.
- A new `Repeater` component for Forms has been added.
- Icons now use the `Filament\Support\Icons\Heroicon` Enum by default. Other options are available and documented.

### Organize Component Classes Structure
- Schema components: `Schemas/Components/`
- Table columns: `Tables/Columns/`
- Table filters: `Tables/Filters/`
- Actions: `Actions/`

### Form Components in v4
- Use `Filament\Forms\Components` for form fields like TextInput, Select, Textarea, etc.
- Use `Filament\Schemas\Components` for layout components like Grid, Section, Fieldset, Tabs, Wizard.

### Actions in v4
```php
use Filament\Actions\Action;

Action::make('send')
    ->label('Send Email')
    ->icon(Heroicon::OutlinedPaperAirplane)
    ->requiresConfirmation()
    ->action(fn (Model $record) => $record->sendEmail());
```

### Table Actions in v4
```php
use Filament\Actions\Action;
use Filament\Actions\DeleteAction;
use Filament\Actions\EditAction;

public static function table(Table $table): Table
{
    return $table
        ->actions([
            EditAction::make(),
            DeleteAction::make(),
            Action::make('approve')
                ->action(fn (Model $record) => $record->approve()),
        ]);
}
```

### Icons in v4
```php
use Filament\Support\Icons\Heroicon;

// Instead of string icons
->icon(Heroicon::OutlinedHome)
->icon(Heroicon::SolidUser)
```

### Testing Filament 4 Resources
```php
use function Pest\Livewire\livewire;

it('can list records', function () {
    $records = Model::factory()->count(5)->create();

    livewire(ListModels::class)
        ->assertCanSeeTableRecords($records);
});

it('can create a record', function () {
    livewire(CreateModel::class)
        ->fillForm([
            'name' => 'Test Name',
        ])
        ->call('create')
        ->assertHasNoFormErrors();

    expect(Model::query()->where('name', 'Test Name')->exists())->toBeTrue();
});

it('can edit a record', function () {
    $record = Model::factory()->create();

    livewire(EditModel::class, ['record' => $record->getRouteKey()])
        ->fillForm([
            'name' => 'Updated Name',
        ])
        ->call('save')
        ->assertHasNoFormErrors();

    expect($record->refresh()->name)->toBe('Updated Name');
});
```


=== .ai/core rules ===

## Filament
- Filament is used by this application, check how and where to follow existing application conventions.
- Filament is a Server-Driven UI (SDUI) framework for Laravel. It allows developers to define user interfaces in PHP using structured configuration objects. It is built on top of Livewire, Alpine.js, and Tailwind CSS.
- You can use the `search-docs` tool to get information from the official Filament documentation when needed. This is very useful for Artisan command arguments, specific code examples, testing functionality, relationship management, and ensuring you're following idiomatic practices.
- Utilize static `make()` methods for consistent component initialization.

### Artisan
- You must use the Filament specific Artisan commands to create new files or components for Filament. You can find these with the `list-artisan-commands` tool, or with `php artisan` and the `--help` option.
- Inspect the required options, always pass `--no-interaction`, and valid arguments for other options when applicable.

### Filament's Core Features
- Actions: Handle doing something within the application, often with a button or link. Actions encapsulate the UI, the interactive modal window, and the logic that should be executed when the modal window is submitted. They can be used anywhere in the UI and are commonly used to perform one-time actions like deleting a record, sending an email, or updating data in the database based on modal form input.
- Forms: Dynamic forms rendered within other features, such as resources, action modals, table filters, and more.
- Infolists: Read-only lists of data.
- Notifications: Flash notifications displayed to users within the application.
- Panels: The top-level container in Filament that can include all other features like pages, resources, forms, tables, notifications, actions, infolists, and widgets.
- Resources: Static classes that are used to build CRUD interfaces for Eloquent models. Typically live in `app/Filament/Resources`.
- Schemas: Represent components that define the structure and behavior of the UI, such as forms, tables, or lists.
- Tables: Interactive tables with filtering, sorting, pagination, and more.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Raffi-001/kognizecrm](https://github.com/Raffi-001/kognizecrm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
