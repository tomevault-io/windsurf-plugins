---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`ebbbang/laravel-mailroom` — a standalone Laravel package (namespace `Ebbbang\Mailroom\`, PSR-4 from `src/`). It registers a `mailroom` mail transport that captures outgoing mail to the database plus a disk, and serves a Blade mailbox UI at `/mailroom`.

There is no host application. Development runs against [Orchestra Testbench](https://packages.tools/testbench): `composer install` scaffolds a skeleton app under `vendor/orchestra/testbench-core/laravel`, and `workbench/` holds the demo app used by `composer serve`.

## Commands

```bash
composer test          # test:parallel then test:isolated -- the full suite
composer test:parallel # everything except the publishes-files group, via paratest
composer test:isolated # just that group, single process
composer test:serial   # the whole suite through phpunit, when parallel output is hard to read

composer lint          # rector then pint -- rewrites files
composer lint:check    # both in dry-run; this is what CI runs

composer serve         # build, create sqlite db, migrate, seed, serve the demo at :8000/mailroom
composer seed          # wipe and reseed the demo mailbox
```

Running one test or one file:

```bash
vendor/bin/phpunit --filter=it_captures_a_raw_message
vendor/bin/phpunit tests/Feature/ForwardMessageTest.php
```

Use `phpunit` rather than `paratest` for a single test — paratest exists only to parallelise the full run.

### Why `composer test` is two passes

Testbench gives every parallel worker the *same* skeleton application, so a test that writes into it corrupts the others. Tests that publish config, rewrite `.env`, or touch anything under `config_path()` carry `#[Group('publishes-files')]` and run alone afterwards. Add that attribute to any new test that writes into the skeleton — otherwise the symptom is an unrelated test failing intermittently on a different worker.

## Architecture

### The capture path

```
MAIL_MAILER=mailroom
  → MailroomServiceProvider::registerTransport()   callAfterResolving('mail.manager')
  → TransportFactory::make()                       throws if mailroom.enabled is false
  → MailroomTransport::send()                      terminal: nothing is relayed onward
  → MessageRecorder::record()                      Symfony message → row + blobs
  → MailroomMessage row  +  RawMessageStore files
  → MessageStored event
```

`MailroomTransport` implements `TransportInterface` directly rather than extending Symfony's `AbstractTransport`, since there is no endpoint to talk to — it is modelled on `Illuminate\Mail\Transport\ArrayTransport`. Capturing at the transport layer (not via a `MessageSent` listener) is what gives access to the finished MIME, attachments and all.

Two subtleties in `MessageRecorder` that are easy to break:

- `getPreparedHeaders()` returns a clone and mints a *fresh* Message-ID and Date when absent. It is called once and the result reused for both the stored `.eml` and the database row; calling it twice makes the row describe a message that does not exist.
- Stream-backed bodies are resolved to strings before MIME generation, since generating would consume the stream and leave nothing for the row.

### Storage is split by design

Metadata and bodies live in the database so the list stays fast; raw MIME and attachment bytes go to a disk so the table stays small. `RawMessageStore` is the single seam to the filesystem, with everything for one message under `{path}/{uuid}/` — so deleting a message is one `deleteDirectory()` and can never orphan a part.

### The deletion invariant

**Every delete must go through model events**, because that is what removes the blobs alongside the row. Consequences already baked in, and worth preserving:

- `MailroomMessage` uses `Prunable`, deliberately not `MassPrunable`.
- `MessageController::clear()` chunks and calls `delete()` per model, then sweeps the directory.
- `migrate:fresh` fires no model events, so `FlushStorageOnDatabaseRefresh` listens for `DatabaseRefreshed` — and only flushes when the refreshed connection matches `mailroom.database.connection`.
- That listener is **not registered during tests**: `RefreshDatabase` runs `migrate:fresh` before a test can call `Storage::fake()`, so it would delete the developer's real captured mail.

### The enabled gate

`Mailroom::enabled()` (config `mailroom.enabled`, off in production by default) gates four separate things, and each is a deliberate choice:

| Gated | Why |
|---|---|
| Route registration | `/mailroom` 404s rather than 403s — no route, no surface |
| `loadMigrationsFrom` | a production deploy gains no tables (this is also why migrations are **not publishable**) |
| `TransportFactory::make()` | throws at construction, so a misconfigured mailer fails loudly instead of swallowing mail |
| Schedule registration | never adds work to an application that did not ask |

### Access control


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ebbbang/laravel-mailroom](https://github.com/ebbbang/laravel-mailroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
