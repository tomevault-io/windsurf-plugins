---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

See [AGENTS.md](./AGENTS.md) — it is the source of truth for agent configuration in this repo.

## House rules

- **The application is `src/`.** The repository root holds only the Docker compose files and
  `.docker/`. Every path in this document is relative to `src/`.
- **Do not write long comments.** Much of the existing tree carries essay-length docblocks
  explaining the defect a line prevents. Do not extend that style and do not add more of it: a
  short comment where the code cannot say it itself, and nothing else. The reasoning belongs in
  the commit message, the ADR or the issue.
- **No annotations in commits or pull requests.** No `Closes owner/repo#123`, no
  `Co-Authored-By`, no "generated with" footer, no trailer of any kind. Subject and body only.
- **Claiming an issue means claiming it on the board.** Any issue picked up that is on
  [project 10](https://github.com/orgs/liberusoftware/projects/10) moves to **In Progress** and is
  assigned to the maintainer *before* work starts, not after:

  ```bash
  ITEM=$(gh project item-list 10 --owner liberusoftware --format json --limit 200 \
    --jq '.items[] | select(.content.number == <issue>) | .id')
  gh issue edit <issue> --repo liberusoftware/ecommerce-laravel --add-assignee @me
  gh project item-edit --project-id PVT_kwDOCXeRJc4BfHfc --id "$ITEM" \
    --field-id PVTSSF_lADOCXeRJc4BfHfczhZcsso --single-select-option-id 47fc9ee4
  ```

  Ids, the `read:project` scope requirement and the `Done` option id are in
  [`docs/agents/issue-tracker.md`](./docs/agents/issue-tracker.md#project-board-10).

## Commands

```bash
composer lint          # pint --test
composer lint:fix      # pint
composer analyse       # phpstan, whole tree
composer test          # php artisan test
composer check         # all three, in that order

php artisan test --filter=CheckoutServiceTest          # one test class
php artisan test tests/Feature/CheckoutServiceTest.php # one file
php artisan test --testsuite=Unit                     # one suite

npm run dev            # vite
npm run build
```

Tests are PHPUnit classes extending `Tests\TestCase`, on sqlite `:memory:`; `phpunit.xml` holds the
env the suite needs (including the social-login provider list, which production leaves empty).

**Neither PHP nor Composer exists on this machine, so none of the above runs from a bare shell.
They run in Docker.** `docker run --rm composer:2` and `php:8.5-cli` resolve Packagist and run
install, Pint, PHPStan, Pest and `pecl install pcov` for coverage — verified 2026-08-17, and used
for every gate of wave 17 before anything was pushed.

The reason this was believed impossible is worth keeping, because it is the same shape as the
Composer note in `docs/MIGRATION_PLAN.md` wave 0. PHP's libcurl is built against c-ares, which
ignores the `options use-vc` that makes `curl`, `git` and `gh` work here. That is a property of the
**host PHP binary**, not of the machine or its network, and a container brings its own. The
inference from *the host PHP cannot resolve* to *nothing here can run a suite* was never checked,
and it stood for sixteen waves because the fallback worked: pushing to a runner is a slower loop,
not a broken one, so nothing ever failed in a way that made anybody re-read the premise.

**GitHub Actions remains the authority.** A local pass is not a result until CI agrees — push a
branch and open a PR, which is what fires `tests.yml` (and `lint.yml`) here and in every module
repo. Never report a gate you have seen pass in only one of the two places.

Two static gates, scoped differently on purpose: **Pint is a ratchet** — CI checks only the PHP
files a PR touches, so editing a long-unformatted file means adopting its formatting in the same
commit. **PHPStan is whole-tree** at level 0 with no baseline; `phpstan.neon` records the level
ladder and why level 8 is not reachable without Larastan.

## Architecture

**A request finds its merchant by hostname.** `ResolveChannel` middleware resolves the host through
`ChannelResolver` to a `Channel`; an unresolved host is a 404 with no default-merchant fallback. A
`Channel` belongs to a `Store`, a `Store` to a `Team` — the tenant boundary, inherited from
Jetstream. Commerce tables scope on `store_id`; everything else on `team_id`.
[`CONTEXT.md`](./CONTEXT.md) defines those five words and is worth reading before using any of them.

**Tenancy is enforced by global scopes, never at the call site.** `App\Traits\IsStoreScoped` and
`App\Traits\IsTenantModel` carry it. The ways out are deliberate and few:
`withoutGlobalScope('store')`, or `StoreContext::acrossAllStores()` for work that is about a person
rather than a shopfront — a GDPR export or erasure narrowed to one store is a wrong answer, not a
partial one. Filament panels are outside this stack: resources are Team-scoped by Filament tenancy,
and the store scope covers what that does not reach (relation managers, widgets, bare
`Model::query()`).

**Two Filament panels**, `app/Providers/Filament/AdminPanelProvider.php` (platform) and
`AppPanelProvider.php` (merchant, Team tenancy).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liberusoftware/ecommerce-laravel](https://github.com/liberusoftware/ecommerce-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
