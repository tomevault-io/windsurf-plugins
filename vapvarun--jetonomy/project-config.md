---
trigger: always_on
description: **Every release zip must be produced by `bin/build-release.sh`.** No exceptions.
---

# Jetonomy — WordPress Forum Plugin

## Build Rule (enforced)

**Every release zip must be produced by `bin/build-release.sh`.** No exceptions.

- Why: on 1.3.5 a stale Desktop zip (built before the critical bootstrap fix was committed) reached the GitHub release and took a customer's live site down. The release agent trusted "zip already exists" instead of rebuilding.
- What the script guarantees:
  1. Clean-tree gate (`--allow-dirty` only for local dev)
  2. Version triangulation — Version header, constant, and readme Stable tag must match
  3. Production composer install in staging (`--no-dev --optimize-autoloader`)
  4. `php -l` on every staged PHP file
  5. **Smoke test** — boots the plugin through `plugins_loaded` + `init` in a minimal WP stub (`tools/wp-stubs.php` + `tools/smoke-test.php`), catching load-time fatals like the 1.3.5 `Jetonomy\table()` bug
  6. Zip → re-extract to scratch → re-run smoke test (catches zip corruption)
- Never attach a pre-existing zip to a release. Always rebuild from the tagged commit.
- Pro: `bin/build-release.sh` additionally enforces the lockstep rule — fails if Pro's version doesn't match free's.

## Pre-Commit Rule (enforced)

**Every commit is locally gated by `.githooks/pre-commit`.** The hook runs PHPStan on the full tree (honours the baseline) and WPCS on staged PHP files before the commit lands. Failures block the commit so red X's never reach the public history.

- Install: `composer install` auto-configures it via `post-install-cmd` (sets `core.hooksPath .githooks`). For existing clones, `composer run hooks:install` does the same.
- Skip (emergencies only): `git commit --no-verify` — but fix it in the next commit.
- Target latency: under 30 seconds. If the hook gets slower, split the check or move it to pre-push.

## Release Rule (enforced)

**Free and Pro always ship with the same `x.y.z` version number.** No exceptions.

- Every release bumps `jetonomy` AND `jetonomy-pro` together, even if one side has no user-facing changes — the side with no changes gets a "Compatibility: Aligned with Jetonomy x.y.z" entry in its readme.
- `JETONOMY_VERSION`, `jetonomy.php` `Version:` header, and `readme.txt` `Stable tag:` must all match the corresponding Pro constants and headers.
- CI fails fast if the two versions drift.
- Rationale: pairing simplifies support ("what version are you on?"), EDD updater routing, and the release checklist — no cognitive load deciding which plugins need which bump.

## Quick Reference
- **Type**: WordPress Plugin (forum, Q&A, ideas, social feed)
- **PHP**: 8.1+ required
- **WP**: 6.7+ required
- **Namespace**: `Jetonomy\`
- **Table prefix**: `jt_` (22 custom tables)
- **REST API**: `jetonomy/v1` (42 endpoints, 15 controllers)

## Architecture
- **Database**: Custom MySQL tables via `dbDelta()` — NOT WordPress CPTs
- **Models**: `includes/models/` — all extend abstract `Model` class
- **Permissions**: 3-layer system (WP Caps → Space Roles → Trust Levels)
- **API**: `includes/api/` — all extend `Base_Controller` → `WP_REST_Controller`
- **Frontend**: PHP templates + WP Interactivity API + CSS Custom Properties
- **Adapters**: Universal adapter pattern for membership, search, real-time, email

## Key Files
| Path | Purpose |
|---|---|
| `jetonomy.php` | Main plugin file, constants, bootstrap |
| `includes/class-jetonomy.php` | Singleton, activation, dependency loading |
| `includes/class-router.php` | URL rewrite rules for /community/* |
| `includes/class-template-loader.php` | Template resolution with theme overrides |
| `includes/db/class-schema.php` | All 21 table definitions |
| `includes/db/class-migrator.php` | Version-based schema migrations |
| `includes/models/` | 15 model classes (Category, Space, Post, Reply, Vote, etc.) |
| `includes/permissions/class-permission-engine.php` | 3-layer permission resolver |
| `includes/trust/` | Trust levels (0-5), reputation calculator, auto-evaluator |
| `includes/api/` | 12 REST API controllers |
| `includes/adapters/` | 4 interfaces + WP Roles, Polling, wp_mail, MemberPress, PMPro adapters |
| `includes/notifications/class-notifier.php` | Event-driven notification dispatcher |
| `includes/import/` | bbPress + wpForo import tools |
| `templates/` | 12 views + 6 partials (theme-overridable) |
| `assets/css/jetonomy.css` | Theme-adaptive CSS (inherits from theme.json) |
| `assets/js/view.js` | Interactivity API store (voting, sorting, polling) |

## Documentation
- **Design Spec**: `docs/specs/2026-03-17-jetonomy-forum-plugin-design.md`
- **Implementation Plans**: `docs/plans/`
- **Design Prototypes**: `docs/prototype/` (open HTML files in browser)

## URL Structure
```
/community/                     → Home
/community/category/:slug/      → Category
/community/s/:slug/             → Space (topic listing)
/community/s/:slug/t/:slug/     → Single post + replies
/community/s/:slug/new/         → New post in space
/community/s/:slug/members/     → Space members
/community/s/:slug/roadmap/     → Space roadmap (ideas)
/community/u/:login/            → User profile
/community/u/:login/edit/       → Edit profile
/community/tag/:slug/           → Tag view
/community/search/              → Search
/community/leaderboard/         → Leaderboard

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vapvarun/jetonomy](https://github.com/vapvarun/jetonomy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
