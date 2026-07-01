---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Community-maintained PHP API client for the Anthropic API (Claude AI). Namespace: `Anthropic\`. Requires PHP 8.2+. Modeled on [openai-php/client](https://github.com/openai-php/client), so when in doubt about patterns or conventions, that repo is a useful reference.

## Commands

```bash
composer install              # Install dependencies
composer lint                 # Fix code style (Pint)
composer test                 # Run ALL checks (lint, types, type-coverage, unit)
composer test:unit            # Run unit tests only (Pest)
composer test:types           # Run PHPStan (level max)
composer test:type-coverage   # Verify 100% type coverage
composer test:lint            # Check code style without fixing
```

Run a single test file:
```bash
./vendor/bin/pest tests/Resources/Messages.php
```

Run a single test by name:
```bash
./vendor/bin/pest --filter="test name substring"
```

## Workflow: Implementing New Features

When asked to develop a new feature, work on a roadmap item, or add support for a new API capability, **always research first, implement second**:

1. **Read the official Anthropic API documentation.** Fetch the relevant pages from `platform.claude.com/docs/` using WebFetch. Understand the exact request schema, response schema, streaming event format, and edge cases. Don't guess or rely on training data alone; the docs may have changed.
2. **Examine existing code patterns.** Read the relevant resource, response DTOs, and tests to understand how similar features are already implemented. Match those patterns exactly.
3. **Build test fixtures from documentation examples.** Never fabricate fixture values. Extract the exact JSON request/response examples from the docs and use those values verbatim (strings, IDs, signatures, text). This way, tests passing means the code actually handles real API responses correctly. For values the docs truncate with "...", drop the trailing dots but keep the rest as-is. If the docs only show a partial response (e.g., just the `content` array), compose the outer envelope from existing fixture patterns.
4. **Implement and test.** Write the code, then run `test:unit`, `test:types`, and `test:lint`.
5. **Update the README** with usage examples that match the documented API, and update this CLAUDE.md if the new feature introduces patterns worth preserving.
6. **Mirror the change in the Laravel wrapper docs.** See the "Keeping the Laravel wrapper docs in sync" section below. Any feature touching user-facing response shapes, request parameters, or control-flow stop reasons must be reviewed against the Laravel package docs.

## Keeping the Laravel wrapper docs in sync

The Laravel wrapper package (`mozex/anthropic-laravel`) carries its own documentation that intentionally does NOT duplicate this package's docs page-for-page. Instead the wrapper docs act as a **shortened, Laravel-flavored layer** with a footer link back to the equivalent page on `mozex.dev/docs/anthropic-php/v1/...` for full detail.

After every documentation change in this package, open the matching Laravel wrapper page and decide whether to mirror, adapt, or defer. Do not skip this step.

### Page mapping

| PHP doc | Laravel doc |
|---------|-------------|
| `docs/usage/messages.md` | `docs/usage/messages.md` |
| `docs/usage/tool-use.md` | `docs/usage/tool-use.md` |
| `docs/usage/server-tools.md` | `docs/usage/server-tools.md` |
| `docs/usage/models.md` | `docs/usage/models.md` |
| `docs/usage/streaming.md` | `docs/usage/streaming.md` |
| `docs/usage/thinking.md` | `docs/usage/thinking.md` |
| `docs/usage/citations.md` | `docs/usage/citations.md` |
| `docs/usage/batches.md` | `docs/usage/batches.md` |
| `docs/usage/token-counting.md` | `docs/usage/token-counting.md` |
| `docs/usage/completions.md` | `docs/usage/completions.md` |
| `docs/reference/meta-information.md` | `docs/reference/meta-information.md` |
| `docs/reference/error-handling.md` | `docs/reference/error-handling.md` |
| `docs/reference/testing.md` | `docs/reference/testing.md` |
| `docs/reference/configuration.md` | `docs/reference/configuration.md` |

### What belongs in the Laravel docs vs what to defer

**Add (or update) on the Laravel side when:**
- The feature has a natural Laravel idiom — `Log`, `Cache`, `Queue`/`ShouldQueue`, `Storage`, Eloquent, `auth()`, `config()`, controllers, form requests, middleware.
- An existing Laravel example would be subtly wrong without the new field. The `tool_use` dispatcher pattern needing a `$block->caller?->type === 'direct'` filter is the canonical example — miss that and users ship a duplicate-execution bug.
- The feature changes control flow users must handle — new `stop_reason` values, refusal responses, pause-and-resume, rate-limit headers they should throttle on. These usually deserve a code snippet, not just a pointer.
- The feature is something a Laravel dev will reach for in normal app flow (persisting `container_upload` file IDs on an Eloquent model, caching the model list, etc.).

**Defer to the PHP docs (just ensure the footer link exists) when:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mozex/anthropic-php](https://github.com/mozex/anthropic-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
