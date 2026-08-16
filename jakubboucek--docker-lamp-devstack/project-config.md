---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Source for the prebuilt Docker images `jakubboucek/lamp-devstack-php` and `jakubboucek/lamp-devstack-mysql` — a LAMP dev stack for local PHP development mirroring shared-hosting environments. The repo builds, tests, and pushes images to Docker Hub; end users only copy `docker-compose.yml` into their projects.

The project is maintained primarily to serve the maintainer's own development workflow and is published because others found it useful. Its feature set is intentionally opinionated: components and extensions are included only when the maintainer actually needs them, and deliberately omitted otherwise. Any apparent imbalance in the package scope is a design choice, not an oversight. Suggestions and critical feedback on the scope are welcome — but never add new components or extensions on your own initiative; always consult the maintainer first.

## Build commands

All build scripts follow the same four-phase pattern (pull → build → test → push), each phase skippable via ENV flags: `NO_PULL=1`, `NO_BUILD=1`, `NO_TEST=1`, `NO_PUSH=1`, `NO_ASSETS=1`.

```bash
./build-php.sh                # build all current PHP versions (8.2–8.5, apache + cli variants)
./php/build-php-8.5.sh        # build a single PHP version (apache + debug variants)
./php/build-php-8.5-cli.sh    # build a single CLI variant
./php/legacy/build-php-legacy.sh  # legacy PHP versions (many disabled — Debian repos shut down)
./build-mariadb.sh            # build all MariaDB versions
./prepare-assets.sh           # download + gzip-compress pie.phar into php/build/ (needed before PHP builds)
./check-pulls.sh              # pull all upstream base images to check for updates
```

Common examples: `NO_PUSH=1 ./build-php.sh` (local build only), `NO_PULL=1 NO_BUILD=1 ./build-php.sh` (push pre-built images).

## Tests

```bash
./test-php.sh                 # run test suite inside every current PHP image (8.2–8.5, apache + cli)
```

Tests live in `tools/tests/` (Nette Tester, one `*Test.php` per PHP extension in `tools/tests/Extensions/`). To run in a single image:

```bash
docker run --rm -it -v "$(pwd)":/app -w /app jakubboucek/lamp-devstack-php:8.5 composer -d tools/ test
```

`tools/compute-minimal-ext-deps.php` is a build helper: run inside an image to compute the minimal runtime package list (`apt-mark` command) that replaces `-dev` packages in Dockerfiles — use it when changing extension deps.

## Architecture

- **`php/Dockerfile-<ver>`** — Apache image based on `php:<ver>-apache-trixie`. Installs extensions via `docker-php-ext-install` plus PIE (`pie install`) for PECL-style extensions (memcached). Dev packages (`EXTENSION_DEV_DEPS`) are installed, extensions compiled, then dev packages purged, keeping only `EXTENSION_RUNTIME_DEPS` (`apt-mark manual`) — this keeps images small. Installs Composer, moves DocumentRoot to `/var/www/html/www`, and makes Apache port configurable via `PORT`.
- **`php/Dockerfile-<ver>-debug`** — layered ON TOP of the published `jakubboucek/lamp-devstack-php:<ver>` image (not the upstream base); adds Xdebug + SPX profiler via PIE.
- **`php/Dockerfile-<ver>-cli`** — CLI variant from `php:<ver>-cli-trixie`.
- **`php/legacy/`** — Dockerfiles for EOL PHP versions (5.4–8.1); mostly unmaintained, several no longer buildable.
- **`mysql/Dockerfile-<ver>`** — thin layer over official `mariadb`, adds `mysql-unicode.cnf` (utf8mb4 default) and `mysql-max-packet.cnf`.
- **Runtime config via ENV**: `php/core.ini` and `opcache*.ini` reference environment variables (`${PHP_MEMORY_LIMIT}` etc.), so users tune php.ini directives at container start without rebuilding. When adding a configurable directive: add it to the ini file, add the `ENV` default in every current Dockerfile, and document it in README.md.

## Versioning and release conventions

- Image tags are cascaded SemVer (`latest` / `8` / `8.5` / `8.5.x`); the patch version is read at build time from the built image (`PHP_RELEASE_VERSION`). MariaDB patch releases are pinned as `MARIADB_RELEASE=<n>` variables in `build-mariadb.sh` — updating a MariaDB version means bumping that number.
- Tag suffixes: `-debug` (Xdebug + SPX), `-cli`, `-legacy` (EOL PHP), `-rc` (pre-releases), `lts` (current MariaDB LTS).
- **Automation asymmetry**: the PHP build is fully automated — a complete rebuild of a new upstream release only requires running `build-php.sh`. The MariaDB build is more primitive by design: exact versions are hard-coded in `build-mariadb.sh`, so every new upstream release requires a code change; once updated, the (re)build itself runs fully automatically.
- **Release workflow**: every patch-level version bump is submitted as a PR and merged into `master` only after the images have been built and verified. Look at merged PRs in the git history for the established format.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakubboucek/docker-lamp-devstack](https://github.com/jakubboucek/docker-lamp-devstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
