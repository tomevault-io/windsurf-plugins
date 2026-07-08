---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
composer run dev                     # Start Laravel server, queue, Pail logs, and Vite concurrently

# PHP tests (SQLite in-memory, RefreshDatabase on every test)
composer run test                    # All tests (clears config cache first)
composer run test:feature            # Feature suite only (parallel)
composer run test:integration        # Integration suite only (parallel)
composer run test:unit               # Unit suite only (parallel)
composer run test:types              # PHPStan level 10 (covers app/ + tests/)
php artisan test tests/Feature/PlannedMeal/PlannedMealStoreTest.php
php artisan test --filter "stores a planned meal"

# Frontend checks
pnpm types        # TypeScript type check
pnpm lint         # ESLint with auto-fix
pnpm format       # Prettier

# Code quality
composer run lint                    # Laravel Pint (PHP auto-formatter)

# Code generation — run after changes
php artisan wayfinder:generate       # Regenerate resources/js/actions/ after route changes
php artisan typescript:transform     # Regenerate TS types after #[TypeScript] Data class changes
```

## Stack

- **Backend**: Laravel 12, PHP 8.2+, Inertia.js, Pest
- **Frontend**: React 19, TypeScript, Tailwind CSS v4, Radix UI, DaisyUI, pnpm
- **Auth**: Laravel Fortify
- **Permissions**: Spatie Laravel Permission (team-scoped roles)
- **Data/DTOs**: Spatie Laravel Data + TypeScript Transformer
- **Routes → TS**: Laravel Wayfinder
- **AI**: OpenAI PHP client — recipe generation uses `gpt-4o-mini`, meal plan generation uses `gemini-3-flash-preview` (both via `app('openai.client')`)

---

## Backend

### Request/Response pipeline

```
HTTP Request
  → Controller (thin: resolves deps, calls action, returns Inertia/redirect)
    → RequestData DTO (Spatie Data, validates via rules())
    → Action::execute(User, ...args) or Action::__invoke(...)
      → Model / DB / Service
    → ResourceData DTO (Spatie Data, passed to Inertia::render())
```

Controllers use the `HasAuthenticatedUser` trait — always call `$this->authenticatedUser()` instead of `auth()->user()`.

### Directory structure

```
app/
  Actions/
    PlannedMeal/   # PlannedMealStoreAction, PlannedMealDestroyAction, PlannedMealWeekQueryAction,
                   # PlannedMealUpdateAction, PlannedMealGeneratePlanAction
    Recipes/       # RecipeStoreAction, RecipeUpdateAction, RecipeDestroyAction,
                   # RecipeAIGenerationAction, RecipeImageAIGenerationAction,
                   # RecipeUploadImageAction, RecipeImageDeleteAction,
                   # RecipeSyncIngredientsAction, RecipeSyncTagsAction,
                   # RecipeSyncMealTimesAction, RecipeSyncStepsAction,
                   # RecipeFiltersAction, RecipeSearchAction, ...
    Workspace/     # WorkspaceStoreAction, WorkspaceUpdateAction, WorkspaceDeleteAction,
                   # WorkspaceGetCurrentAction, WorkspaceInvitationStoreAction,
                   # WorkspaceInvitationAcceptAction, WorkspaceInvitationDeclineAction,
                   # WorkspaceMemberDeleteAction, WorkspaceMemberRoleUpdateAction
  Data/
    Requests/      # Incoming DTOs with rules() for validation
    Resources/     # Outgoing DTOs sent to Inertia (many marked #[TypeScript])
  Enums/
    Unit           # Ingredient units (ml, g, tsp, piece, etc.) — used by AI prompts
  Http/Controllers/
  Models/
  Observers/       # RecipeObserver (deletes image + planned meals on delete)
                   # PlannedMealObserver (syncs shopping lists)
  Policies/        # Bound via #[UsePolicy] attribute on models
  Services/
    AIMealPlanningService   # Generates weekly meal plans via AI function calling
    ShoppingListService
```

### Models & domain

**Key relationships:**

```
User ──< Recipe (user_id, UUID PK)
          ├── BelongsToMany MealTime  (via recipe_meal_time)
          ├── BelongsToMany Ingredient (via recipe_ingredient, pivot: quantity, unit)
          ├── HasMany Step
          └── BelongsToMany Tag (via recipe_tag)

User >─< Workspace (via workspace_users, pivot: joined_at)
Workspace ──< PlannedMeal (workspace_id + user_id + recipe_id + meal_time_id + planned_date)
PlannedMeal ──< ShoppingListPlannedMealIngredient

Workspace ──< WorkspaceInvitation (expires_at, token)
```

**Conventions:**

- `Recipe` uses `HasUuids` (UUID primary key); all other models use auto-increment int IDs.
- Policies are bound with `#[UsePolicy(XxxPolicy::class)]` on the model, not in `AuthServiceProvider`.
- Observers are registered with `#[ObservedBy([XxxObserver::class])]` on the model.
- All datetime columns cast to `immutable_datetime`.

A `PlannedMeal` stores **both** `user_id` (who planned it) and `workspace_id` (the shared context). Storing a duplicate (same user + workspace + recipe + meal_time + date) increments `serving_size` instead of creating a new row.

### Workspaces & permissions

The active workspace is stored in session (`current_workspace_id`). `WorkspaceGetCurrentAction` reads it and falls back to the user's personal default workspace.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lionelp-dev/Mealo](https://github.com/lionelp-dev/Mealo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
