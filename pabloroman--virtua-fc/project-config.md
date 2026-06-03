---
trigger: always_on
description: VirtuaFC is a football manager simulation game built with Laravel 12. Players manage Spanish football teams (La Liga/Segunda División) through seasons, handling squad selection, transfers, and competitions including the Copa del Rey and European competitions (Champions League, Europa League, Conference League).
---

# CLAUDE.md

## Project Overview

VirtuaFC is a football manager simulation game built with Laravel 12. Players manage Spanish football teams (La Liga/Segunda División) through seasons, handling squad selection, transfers, and competitions including the Copa del Rey and European competitions (Champions League, Europa League, Conference League).

The frontend uses Blade templates with Tailwind CSS and Alpine.js. The app defaults to Spanish (`APP_LOCALE=es`).

**Stack versions:** PHP 8.5, Laravel 12, PHPUnit 11. Tailwind CSS 4.x (via `@tailwindcss/vite`), Alpine.js 3.x, Vite 7.x, Vitest 4.x. No ESLint/Prettier — only `.editorconfig` (4-space indent, LF, UTF-8) and Laravel Pint for PHP.

## Development Commands

```bash
composer dev                                    # Run all services (server, queue, vite, logs)
php artisan test                                # Run tests
php artisan test --filter=TestClassName          # Run a single test
php artisan app:seed-reference-data             # Seed reference data (--fresh to reset)
php artisan app:simulate-match                  # Simulate a match (debugging)
php artisan app:simulate-season                 # Simulate a full season
php artisan config:clear                        # Clear config cache after changes
./vendor/bin/phpstan analyse                    # Larastan static analysis (level 1)
```

The queue worker must be running for background jobs. `composer dev` handles this via `php artisan queue:listen --tries=1`.

**Do not run tests or static analysis automatically after making changes.** Both run in CI after pushing to a branch. Only run them locally when explicitly asked.

**Game-state debugging commands** (full list in `app/Console/Commands/`):

```bash
php artisan app:diagnose-stuck-game {game}      # Investigate a stalled game
php artisan app:cleanup-games                   # Remove orphaned/abandoned games
php artisan app:refresh-player-templates        # Reseed player biography source
php artisan app:unstick-season-transition       # Unblock a stuck season transition
```

## Testing

- **PHPUnit 11** (no Pest). Tests live in `tests/Unit/` and `tests/Feature/`.
- The base `tests/TestCase.php` sets `protected $connectionsToTransact = ['pgsql']` and calls `$this->withoutVite()` in `setUp()`.
- **Factories use fluent helpers** — prefer them over manual wiring. Examples: `Game::factory()->forTeam($team)->create()`, `GamePlayer::factory()->forTeam($team)->create()`, `Game::factory()->inCompetition($id)->create()`.
- Parallel runs via `paratest` are available (`php artisan test --parallel`) but not the default.
- Static analysis: Larastan at level 1 (`./vendor/bin/phpstan analyse`). Strict where it matters; permissive elsewhere.

## Architecture

### HTTP Layer

Uses invokable single-action classes instead of controllers:

- **Actions:** `App\Http\Actions\*` — form submissions and game commands
- **Views:** `App\Http\Views\*` — prepare data for Blade templates
- **Auth:** Laravel Breeze controllers in `App\Http\Controllers\Auth\`

**Views and Actions must stay thin.** They only orchestrate: validate input, call a service, return a response. Business logic, database queries, and data transformations belong in service classes (`app/Modules/*/Services/`). Never put domain logic in a View or Action.

**Route wiring** (`routes/web.php`): routes bind invokable Actions/Views directly — `Route::get('/manager/{username}', ShowManagerProfile::class)`. No controller classes for game flows. Route names use dot notation (`leaderboard.teams`, `tournament-summary.show`). Game-scoped routes go behind the `game.owner` middleware (defined in `bootstrap/app.php`, implemented by `App\Http\Middleware\EnsureGameOwnership`). Any new route that operates on a specific game must be inside that middleware group.

### Modular Monolith

Domain logic is organized into modules under `app/Modules/`, each with services, contracts, DTOs, and events. Conceptual mechanics for many of these modules are documented in `docs/game-systems/` (index: `docs/game-systems/README.md`) — cross-references are noted below.

| Module | Purpose | Key services | Deep dives in `docs/game-systems/` |
|--------|---------|-------------|------------------------------------|
| **Match** | Match simulation engine | `MatchSimulator`, `MatchdayService`, `CupTieResolver`, handlers | `match-simulation.md`, `matchday-advancement.md` |
| **Lineup** | Tactical layer | `LineupService`, `SubstitutionService`, `FormationRecommender` | — |
| **Player** | Player lifecycle | `PlayerDevelopmentService`, `PlayerConditionService`, `PlayerValuationService`, `InjuryService`, `PlayerRetirementService` | `player-development.md`, `player-abilities.md`, `player-potential.md`, `injury-system.md` |
| **Squad** | Squad composition | `PlayerGeneratorService`, `EligibilityService` | `squad-page-redesign.md` |
| **ReserveTeam** | Reserve / B-team and U23 cascades | `ReserveTeamService` | — |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pabloroman/virtua-fc](https://github.com/pabloroman/virtua-fc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
