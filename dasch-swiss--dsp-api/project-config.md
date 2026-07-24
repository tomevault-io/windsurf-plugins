---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DSP-API is the Digital Humanities Service Platform API - a Scala-based REST API for managing semantic data and digital assets in the humanities. 
The project uses ZIO for functional programming, zio-http as HTTP server and tapir for defining endpoints of the API, and integrates with Apache Jena Fuseki triplestore and Sipi media server.

## Naming Conventions

The legacy project name **"Knora"** still appears in the codebase (package `org.knora.webapi`, class names like `KnoraGroupService`). This is acceptable in code — do not rename packages or classes without an explicit refactoring task.

However, **do not use "Knora" in human-readable text**: PR titles, commit messages, documentation, comments, spec files, or learning documents. Use "dsp-api" instead.

## Build System & Commands

### Core Build Tool

- **Primary**: `just` (command runner) - the canonical entry point for build, test, image, CI, and
  local-dev tasks (stack lifecycle, DB init, cleanup); it wraps Bazel. Run `just --list`.
- **Compilation / formatting**: `sbt` - use the `./sbtx` wrapper (still the Scala build of record
  during the Bazel validation window).

### Essential Development Commands

**Testing:**

- Run a single test: `sbt "testOnly *TestClassName*"`
- Run tests in a specific package: `sbt "testOnly org.knora.webapi.slice.admin.*"`
- `sbt test` - Run unit tests
- Integration tests use `latest` Sipi image by default. To use exact git version locally, set `SIPI_USE_EXACT_VERSION=true` or build with `just docker-build-sipi-image`.
- `just test-unit` - Run all pure-JVM unit tests (webapi, ingest, bagit, jwt, shacl-validator)
- `just test-it` - Run integration tests (requires Docker)
- `just test-e2e` - Run end-to-end HTTP API tests (requires Docker)

**Code Quality:**

- `sbt fmt` - Format code with Scalafmt
- `sbt check` - Check code formatting and linting
- `sbt "scalafixAll --check"` - Check Scalafix rules

**Building:**

- `sbt compile` - Compile the project
- `just docker-build` - Build the dsp-api/sipi/ingest Docker images (Fuseki excluded)
- `just docker-build-dsp-api-image` - Build only the API Docker image

**Local Development Stack:**

- `just stack-start` - Start the full stack (Fuseki, Sipi, API)
- `just stack-start-dev` - Start stack without API (for development)
- `just stack-stop` - Stop the stack
- `just stack-init-test` - Initialize with test data

### Bazel & the Nix dev shell

All four container images (`knora-sipi`, `knora-api`, `dsp-ingest`, `apache-jena-fuseki`) build with
**Bazel** (`rules_oci`).
Bazel is provided through a **Nix dev shell** (`flake.nix`) that puts `bazel` (a bazelisk wrapper;
the version is pinned in `.bazelversion`), a JDK 25, `just`, and `crane` on `PATH`.

- **Enter the shell:** with `direnv` it loads automatically on `cd` into the repo (`.envrc` runs `use flake`; run `direnv allow` once). Without direnv, prefix commands with `nix develop --command`, e.g. `nix develop --command just docker-build-sipi-image`.
- `just docker-build-sipi-image` / `just docker-build-dsp-api-image` / `just docker-build-ingest-image`
  build the image and load it into the local Docker daemon (`:latest` plus the git-describe
  version tag); each runs `bazel run //modules/<sipi|webapi|ingest>:load`. `just docker-publish-*`
  variants build + push the multi-arch image via `bazel run //modules/<m>:push`.
- `bazel build //modules/webapi:image_amd64` / `//modules/ingest:image_amd64` - build the knora-api /
  dsp-ingest images directly; `bazel run //modules/webapi:load` / `//modules/ingest:load` loads
  them into the local Docker daemon at the same `:latest` tags `docker-compose.yml` uses.
- Fuseki's recipes (`just docker-build-fuseki-image`/`just docker-publish-fuseki-image`) build and
  publish the fuseki image with Bazel (`//modules/fuseki:load`/`:push`), same as the other three.
- CI runs entirely through `just` and `bazel test`, pointed at the shared **NativeLink RBE backend**
  (`dasch-remotebuild-prod-01`) for a remote cache + executor. See `docs/development/dsp-api-rbe.md`.
  sbt is retained only for the formatting check (`just check`) and a temporary tag-drift gate
  (`just check-docker-image-tag`), both removable once validation closes.

## Architecture

### Module Structure

The codebase is organized into several key modules:

**Core Modules:**

- `modules/webapi/` - Main API application
- `modules/bagit/` - BagIt library for creating, reading, and validating BagIt packages (RFC 8493, <https://www.rfc-editor.org/rfc/rfc8493>)
- `modules/testkit/` - Shared test utilities and base classes
- `modules/test-it/` - Integration tests (service/repo/Sipi tests)
- `modules/test-e2e/` - End-to-end HTTP API tests
- `modules/sipi/` - Custom Sipi media server configuration

**Slice Architecture** (`modules/webapi/src/main/scala/org/knora/webapi/slice/`):

- `admin/` - Administrative endpoints (users, groups, projects, permissions)
- `common/` - Shared utilities and base classes
- `infrastructure/` - Cross-cutting concerns (metrics, caching, JWT)
- `lists/` - List management functionality
- `ontology/` - Ontology management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dasch-swiss/dsp-api](https://github.com/dasch-swiss/dsp-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
