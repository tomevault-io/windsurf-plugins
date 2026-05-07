---
trigger: always_on
description: > Place this file at **`.github/copilot-instructions.md`** in the repository root so Copilot (Chat, PR review, code suggestions, and Coding Agent) loads it automatically.
---

# Copilot Instructions — Mailbox for Laravel (Repository‑wide)

> Place this file at **`.github/copilot-instructions.md`** in the repository root so Copilot (Chat, PR review, code suggestions, and Coding Agent) loads it automatically.
>
> See also: `CLAUDE.md` for Claude Code-specific guidance and `ARCHITECTURE.md` for dashboard isolation details.

## Project overview

* This repository is a **Laravel package** that embeds a local email mailbox for development and QA, similar to Mailtrap but self‑contained. It ships a Vue‑powered dashboard, storage drivers, a custom mail transport, and HTTP controllers.
* Primary goals: zero‑config local email capture, a clean UI, and **full automated test coverage** for every class and feature.

## Tech stack & versions (default)

* **PHP**: ^8.3 (strict types, typed properties, readonly where applicable)
* **Laravel**: ^10 | ^11 | ^12 | ^13
* **Testing**: Pest 2/3/4, Laravel test helpers; Coverage target: **90%+ lines**, **80%+ branches**
* **Static analysis**: PHPStan (Larastan) level 5
* **Code style**: Laravel Pint (PSR‑12), EditorConfig
* **Front end**: Vue 3 + Vite, TypeScript (for any new scripts), TailwindCSS v4 with **scoped/prefixed classes** to avoid collisions with host apps
* **UI components**: Reka UI (radix-vue successor)
* **Tooling**: ESLint + Prettier (for JS/TS/Vue), Stylelint (optional)

## How to build & test (for Copilot to follow)

* **Install PHP deps**: `composer install`
* **Install Node deps**: `npm ci`
* **Build assets for the package**: `npm run build`
* **Run tests (fast)**: `./vendor/bin/pest -p`
* **Run tests (coverage)**: `./vendor/bin/pest --coverage --min=90`
* **Static analysis**: `./vendor/bin/phpstan analyse`
* **Lint**: `./vendor/bin/pint -v`

## Architecture (high level)

* **Transport**: `MailboxTransport` captures `SentMessage`, normalizes it, stores payload, optionally decorates another transport. Decoration is declarative via `mailbox.decorate` (mailer name) — when set, capture and real delivery happen with zero user-code changes.
* **Capture & Storage**:

    * `CaptureService` is the high-level API: store/list/find/update/delete/purge. Owns cascading attachment cleanup. Mints canonical ULIDs and provides write-path idempotency via the RFC 822 `message_id` header.
    * `MessageStore` contract (10 methods) — default driver is `sqlite` (`DatabaseMessageStore` against a dedicated SQLite file at `storage/app/mailbox/mailbox.sqlite`). `database` is an alias for bring-your-own-connection users. `file` driver writes JSON on disk.
    * `AttachmentStore` contract (8 methods) — paired with the active `MessageStore` driver. `DatabaseAttachmentStore` for sqlite/database; `FileAttachmentStore` for the file driver. Both return `DTO\StoredAttachment` value objects.
    * `MessageSearch` contract — pluggable search strategy (`DefaultMessageSearch` searches `subject`, `from`, `to`, `html`, `text`).
    * `StoreManager` resolves drivers via config & custom resolvers.
* **DTOs**: `MailboxMessageData`, `AttachmentData`, `StoredAttachment`, `PaginatedMessages` — typed value objects under `src/DTO/`.
* **HTTP**: `MailboxController`, `SendTestMailController`, `ClearMailboxController`, `DeleteMailboxMessageController`, `SeenController`, `PublicAssetController`, `AttachmentController`, `AuthorizeMailboxMiddleware` (route gating).
* **Retention**: `MailboxServiceProvider` registers a daily `mailbox:clear --outdated` via `callAfterResolving(Schedule::class, …)`, triple-guarded by `mailbox.enabled`, `mailbox.retention > 0`, and `mailbox.retention_schedule`.
* **Support**: `MessageNormalizer` (canonical payload + attachment extraction), `CidRewriter` (rewrites inline `cid:` references), `MailboxServiceProvider`, `InstallCommand`, `UpgradeCommand`, `ClearInboxCommand`, `config/mailbox.php`.
* **Testing API**: `src/Testing/` — `InteractsWithMailbox` trait, `MailboxAssertions`, `PendingMailboxMessageAssertion`, plus facade-level `Mailbox::assertSent()` etc.

## Directory & naming conventions

* **Namespaces**: `Redberry\MailboxForLaravel\...`
* **Contracts** under `Contracts/`; **Support** under `Support/`; **DTO** under `DTO/`; **Testing** under `Testing/`
* **HTTP** under `Http/Controllers` and `Http/Middleware`
* **Storage drivers** under `Storage/`
* **Tests** mirror src namespaces: `tests/Unit`, `tests/Feature`, `tests/Commands`, `tests/Architecture`
* **Vue** in `resources/js/`, entry `resources/js/dashboard.js`. Built assets ship to `public/vendor/mailbox/` (NOT `public/build/`).

## Coding standards for Copilot

### PHP & Laravel

* Prefer **value objects**, **enums**, and **DTOs** over loose arrays where practical.
* Always add **return types** and **param types**. No mixed unless unavoidable; document with PHPDoc if union types are complex.
* **Never** call `env()` outside `config/` files. Use `config()` or typed config accessors.
* Avoid facades in core services; depend on **interfaces** and constructor injection; keep controllers thin.
* **Immutable data** where possible (`readonly` properties, new instances vs mutation).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedberryProducts/mailbox-for-laravel](https://github.com/RedberryProducts/mailbox-for-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
