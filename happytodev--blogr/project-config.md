---
trigger: always_on
description: Blogr is a **FilamentPHP v4 plugin** adding a multilingual blog system to Laravel applications. Core stack: PHP 8.3+, Laravel 12, Pest PHP 4.0 (555+ tests). Package version: 0.12.5 (approaching RC1 - Oct 2025).
---

# GitHub Copilot Instructions for Blogr

## Project Overview
Blogr is a **FilamentPHP v4 plugin** adding a multilingual blog system to Laravel applications. Core stack: PHP 8.3+, Laravel 12, Pest PHP 4.0 (555+ tests). Package version: 0.12.5 (approaching RC1 - Oct 2025).

## Mandatory instructions (IMMUABLES ET UNBREAKABLE)
- Always use `cat` to create files
- Always use french to talk to me
- **🚨 RÈGLE ABSOLUE : NEVER EVER present work as complete if ANY tests are failing 🚨**
- **🚨 RÈGLE ABSOLUE : NEVER make a "bilan" or status report with failing tests 🚨**
- **🚨 RÈGLE ABSOLUE : Continue working until 100% of tests pass (all green) 🚨**
- **🚨 RÈGLE ABSOLUE : If blocked, explicitly state what prevents tests from passing and ask for guidance 🚨**
- **The work is NOT done until: `./vendor/bin/pest` shows 0 failed tests**
- If a task creates new tests, those tests MUST pass before claiming completion
- If a task breaks existing tests, those tests MUST be fixed before claiming completion

## Technology Stack Requirements

### FilamentPHP v4 (CRITICAL)
- **Forms**: `Filament\Schemas\Schema` (NOT `Filament\Forms\Form`)
- **Components**: 
  - `Section`: `Filament\Schemas\Components\Section` (NOT `Filament\Forms\Components\Section`)
  - `Repeater`: Use with `->relationship()` for translations (NOT `Tabs\Tab` pattern)
  - Form inputs (TextInput, Select, etc.): Still in `Filament\Forms\Components\*`
- **Actions**: `Filament\Actions\EditAction`, `Filament\Actions\DeleteAction`, `Filament\Actions\DeleteBulkAction`
- **Tables**: `Filament\Tables\Table` (unchanged from v3)
- **Navigation**: Use methods instead of properties
  ```php
  // ❌ Wrong (v3)
  protected static ?string $navigationIcon = 'heroicon-o-document';
  protected static ?string $navigationGroup = 'Content';
  
  // ✅ Correct (v4)
  public static function getNavigationIcon(): ?string {
      return 'heroicon-o-document';
  }
  public static function getNavigationGroup(): ?string {
      return __('Content');
  }
  ```
- **Method signatures**:
  ```php
  public static function form(Schema $schema): Schema { ... }
  public static function table(Table $table): Table { ... }
  ```
- **Translation Pattern** (v4 best practice):
  ```php
  // Use Repeater with relationship() for translations
  Section::make('Translations')
      ->schema([
          Repeater::make('translations')
              ->relationship()
              ->schema([
                  Select::make('locale')->options(['fr' => 'Français', 'en' => 'English']),
                  TextInput::make('title')->required(),
                  MarkdownEditor::make('content')->required(),
              ])
              ->collapsible()
              ->itemLabel(fn (array $state) => strtoupper($state['locale'] ?? 'NEW'))
              ->addActionLabel('Add Translation'),
      ]);
  // ❌ Don't use Tabs\Tab::make() pattern (v3 only)
  ```

### Laravel 12
- Use latest syntax and features
- Leverage new routing capabilities
- Use PHP 8.3+ features (readonly properties, typed properties, etc.)

### Tailwind CSS 4
- Use Tailwind 4 class names and utilities
- Leverage new container queries if needed
- Use modern color palette (not deprecated colors)

### Pest PHP 4.0
- Use `it()` syntax for tests
- Use `expect()` for assertions
- Use `beforeEach()` and `afterEach()` for setup/teardown
- Architecture tests with `arch()` if needed

## Architecture: Translation-First Design

**Critical Pattern**: All content entities use separate translation tables.

### Database Schema Pattern
- **Main tables** (`blog_posts`, `blog_series`, `categories`, `tags`): Non-translatable fields only (IDs, timestamps, user_id, etc.)
- **Translation tables** (`*_translations`): All locale-specific content (title, slug, excerpt, content, seo fields, photos)
- **Unique constraints**:
  - All translations: `[entity_id, locale]` (one translation per locale per entity)
  - Categories/Tags: `[locale, slug]` (slug unique within locale)
  - Posts: `slug` globally unique (no locale prefix needed)
- **Pivot tables**: `blog_post_translation_category`, `blog_post_translation_tag` (link translations, not main entities)

### Model Relationships
```php
// Main model
public function translations() {
    return $this->hasMany(EntityTranslation::class);
}

// Always query through translations:
BlogPost::with('translations')->get(); // NOT BlogPost::all()

// Tags with automatic alphabetical sorting (v0.15.12+)
// The BlogPost model has a getTagsAttribute() accessor that automatically
// sorts tags alphabetically by their translated name in the current locale
$post->tags; // Returns Collection sorted alphabetically
```

### Sorting Multilingual Relations
**Pattern**: Use Eloquent accessors for automatic sorting with translations

```php
// Example: BlogPost tags sorted alphabetically
public function getTagsAttribute($value) {
    if (!$this->relationLoaded('tags')) {
        $this->load('tags.translations');
    }
    $locale = app()->getLocale();
    $tags = $this->getRelationValue('tags'); // Avoid accessor recursion
    return $tags->sortBy(function($tag) use ($locale) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [happytodev/blogr](https://github.com/happytodev/blogr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
