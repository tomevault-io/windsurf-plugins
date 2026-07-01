---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

Workflow for [Alfred](https://www.alfredapp.com) to search GitHub and GitHub Enterprise. Pure PHP (>= 8.2), no framework. Distributed as `github.alfredworkflow` — a zip with the `.alfredworkflow` extension (the format Alfred expects); built via `PharData` in zip mode, downloaded from GitHub Releases.

## Common commands

```bash
composer install          # install dev deps (php-cs-fixer, phpstan, phpunit)
composer cs-fixer         # apply code style (see .php-cs-fixer.dist.php)
composer phpstan          # static analysis (see phpstan.dist.neon)
composer phpunit          # run the test suite (see phpunit.dist.xml)
npm install               # pulls @primer/octicons (only needed when regenerating icons)
bin/create_icons.php      # regenerate icons/*.png from octicons SVGs (needs Imagick + rsvg-convert)
bin/build                 # bundles release into github.alfredworkflow, writes VERSION + CHANGELOG into info.plist
```

## Local hotkeys in info.plist

When the working copy lives inside `Alfred.alfredpreferences/workflows/...`, Alfred writes personal hotkey bindings (`hotkey`, `hotmod`, `hotstring`) directly into `info.plist`. Those are local-only and must not be committed. A pair of git filters keeps them invisible to the index but preserved across checkouts:

- **clean** (`bin/info-plist-clean`) normalizes hotkey values to neutral placeholders on stage/diff, so `git status` only shows real, non-hotkey changes.
- **smudge** (`bin/info-plist-smudge`) re-injects hotkey values from the existing working-tree file when a checkout/pull would otherwise overwrite them with placeholders.

One-time setup per clone:

```bash
git config filter.alfred-plist.clean  ./bin/info-plist-clean
git config filter.alfred-plist.smudge './bin/info-plist-smudge %f'
```

`.gitattributes` wires `info.plist` to the filter; both scripts live in `bin/`. The filter is not marked `required`, so contributors who don't set it up get a passthrough — they'll just see the placeholder values in their working copy after clone, which is the desired state for them.

`bin/build` runs the same clean script on `info.plist` before bundling it into the release zip, so shipped workflows never carry the maintainer's personal hotkeys regardless of the local git config.

## Tests

Tests live under `tests/`, bootstrapped via `tests/bootstrap.php` (loads `src/workflow.php`, which transitively pulls in `item.php`, `fetcher.php`, `curl.php`). PHPUnit ^11.5 (PHP 8.2 compatible) drives them, CI runs the suite on PHP 8.2 – 8.6.

- **Pure unit tests**: `ItemTest` (matching, ranking, XML serialization), `WorkflowTest` (config scoping, enterprise URL derivation, `getApiUrl` query handling, version compare via cached release).
- **Integration tests against a local `php -S` server**: `CurlTest` (status, headers, ETag/304, auth) and `FetcherTest` (cache hit/miss, pagination chain, stale-while-revalidate, field whitelist, `streamUrl`). The server boilerplate (free port, spawn, ready-poll, teardown) is in `HttpServerTestCase`; subclasses just declare a router fixture under `tests/fixtures/`.
- `WorkflowTest` and `FetcherTest` initialize `Workflow` against a per-class tmpdir so the SQLite DB is fresh.

## Entry points and request flow

All shipped PHP lives under `src/`. Alfred calls one of two PHP scripts via `info.plist` script filter / action bindings:

- `src/search.php` — invoked on every keystroke; returns Alfred XML items. The first query token dispatches: `>` → system commands, `@user` → user subcommands, `user/repo …` → repo subcommands, `s …` → API search, `my …` → "my" pages, otherwise default (orgs/starred/subscribed/repos/following).
- `src/action.php` — invoked when the user actions an item. If the arg is a URL, opens it (with a `.git` → `x-github-client://openRepo/…` rewrite). Otherwise the arg starts with `>` (or `e >` for enterprise) and matches a system command (`login`, `logout`, `delete-cache`, `update`, `refresh-cache`, etc.).

`src/server.php` is a tiny built-in PHP web server started on `localhost:2233` during OAuth login (`gh > login`). GitHub's OAuth callback hits it with `?access_token=…`, which it persists and triggers a cache warmup. `Workflow::startServer()` spawns it; `stopServer()` kills it.

## Core architecture

**`Workflow` (src/workflow.php)** — static singleton. Owns the SQLite DB at `$alfred_workflow_data/db.sqlite` (tables: `config`, `request_cache`), holds the access token, base URLs, and the items list. The enterprise flag flips `$baseUrl` / `$apiUrl` / `$gistUrl` and which token key is used (`access_token` vs `enterprise_access_token`).

**`Fetcher` (src/fetcher.php)** — the cache-aware HTTP layer. Two faces of the same class:
- **Static** for synchronous single-shot calls (most common): `Fetcher::requestApi($path)`, `Fetcher::requestUrl($url)`, `Fetcher::streamApi($path)` (returns `Generator`, page-by-page from cache, bounded memory), `Fetcher::requestRaw($url)` (uncached, raw body — used for the update download).
- **Instance** for parallel batch: `$f = new Fetcher(); $f->queueApi($path, $cb)->queueUrl($url, $cb2)->run();` — multiple URLs go through one `curl_multi_*` loop, callbacks fire as responses come in.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gharlan/alfred-github-workflow](https://github.com/gharlan/alfred-github-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
