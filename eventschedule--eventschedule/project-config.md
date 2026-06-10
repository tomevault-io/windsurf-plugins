---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Event Schedule is an open-source platform for sharing events, selling tickets, and bringing communities together. It supports both hosted (SaaS at eventschedule.com) and selfhosted deployments.

## Important Rules

- **Never run tests without asking first** - Tests will empty the database
- **Never run `npm install` without asking first** - Confirm before installing dependencies
- **Never run `composer install` without asking first** - Confirm before installing dependencies
- **Never delete migration files** - They may have already been run on production
- **Use today's date for new migrations** - Migration filenames must use today's date (e.g. `2026_04_15_000000_`), never a future or past date
- **Use "selfhost" not "self-host"** - Always write "selfhost" and "selfhosted" (no hyphen) except for "self-hosting"
- **Keep the sitemap up-to-date** - When adding new pages, add them to `resources/views/sitemap.blade.php`
- **Complete bento grids** - When using bento grids, ensure all cells are filled (especially the bottom right corner)
- **Align card actions to bottom** - In grids of cards/panels with varying content lengths, use `flex flex-col` on the card and `mt-auto` on the bottom element (e.g. links, buttons) so they align across cards
- **Support light and dark mode** - Always consider both light mode and dark mode when working on UI
- **Forward button at the end** - In button pairs (e.g. cancel/submit), place the forward action button at the end (right in LTR, left in RTL)
- **No co-author on commits** - Do not add "Co-Authored-By: Claude" to git commit messages
- **Never use em-dashes** - Use hyphens, "to", or "or" instead of em-dashes (—) in all written content
- **Use "schedule" not "role", "sub-schedule" not "group"** - In the code, `Role` = schedule and `Group` = sub-schedule. Always use "schedule" and "sub-schedule" in UI text and conversations, never "role" or "group"
- **MySQL only** - Only MySQL is supported; do not add SQLite compatibility to migrations or tests
- **Never use CDNs** - Always use local vendor files for JS/CSS libraries. Selfhosted users should not have the app calling external servers.
- **Never add npm dependencies** - Do not use `npm install` to add new packages. Instead, download built files manually and place them in `public/vendor/`.
- **Use `<x-link>` for inline text links** - Always use the `<x-link>` Blade component for inline text links (not navigation or buttons). It provides consistent styling, dark mode support, and an external link icon for `target="_blank"` links.
- **Use `config('app.supported_languages')` for language lists** - Never hardcode language code arrays. Always reference the centralized list in `config/app.php`.
- **Keep Help button mappings up-to-date** - When adding, removing, or moving doc pages, update the anchor map in `app/Utils/HelpUtils.php` so the admin panel Help button links to the correct docs for each section/tab
- **Match docs structure to app layout** - Documentation sections and sub-sections should mirror the app's UI structure (sections, tabs, sidebar items) where it makes sense. This keeps the Help button deep links aligned and makes docs intuitive for users navigating between the app and docs.
- **Keep `translateData` and `console.php` in sync** - Scheduled commands must be registered in both `AppController::translateData()` (hosted cron) and `routes/console.php` (selfhosted scheduler). When adding a new scheduled command, add it to both places with matching frequency.
- **Use toggle switches for boolean settings** - In the admin portal, use `<x-toggle>` (or toggle switch markup for Vue pages) for standalone boolean on/off settings. Reserve plain checkboxes for multi-select lists and "required" indicators.
- **Consistent primary action button sizing** - Primary action buttons in the AP should use `px-4 py-3 text-base` sizing to match `<x-brand-link>` / `<x-secondary-link>` components. Do not use smaller `py-2 text-sm` for standalone call-to-action buttons.
- **Keep doc search index up-to-date** - When adding, removing, or renaming doc sections, update `getDocSearchIndex()` in `MarketingController` so the docs search stays accurate
- **Follower emails are visible on all schedule-owner-facing surfaces** - Schedule owners can see their followers' name and email on the followers tab (`show-admin-followers.blade.php`), the newsletter stats and segment-edit pages, and the dashboard recent-activity feed. Follower emails must NEVER appear on public/guest-facing surfaces (public stats, embed widgets, guest pages) - those do not list individual followers. When a user clicks Follow on the guest portal, a consent modal (`resources/views/partials/follow-consent-modal.blade.php`) discloses that the schedule will see their name and email.
- **Never expose raw exception messages to users** - In catch blocks that handle user-facing responses, catch `QueryException` (and other system exceptions) separately and show a generic error message. Use `report($e)` to send to Sentry. Only show `$e->getMessage()` for intentional business logic exceptions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eventschedule/eventschedule](https://github.com/eventschedule/eventschedule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
