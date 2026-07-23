---
trigger: always_on
description: This file is the repository knowledge base for collaborators and future AI agents working on Spry.
---

# AGENTS

This file is the repository knowledge base for collaborators and future AI agents working on Spry.

It should help a new contributor quickly understand:

- what Spry is
- how the repository is organized
- where to make changes
- how to validate work
- how commits, PRs, changelogs, and releases should be written

## Project Summary

Spry is a Dart server framework centered on filesystem routing and generated runtime output.

The authoring model is:

- application routes live in `routes/`
- global middleware lives in `middleware/`
- scoped middleware and error handlers live inside `routes/` as `_middleware.dart` and `_error.dart`
- build/runtime behavior lives in `spry.config.dart`
- generated runtime output goes to `.spry/` by default

Spry targets multiple runtimes:

- Dart VM
- Node.js
- Bun
- Cloudflare Workers
- Vercel
- Netlify Functions

## Core Mental Model

The core pipeline is:

1. load config
2. scan the project tree
3. build a route tree
4. generate runtime entry files
5. write generated output
6. optionally compile JS for non-Dart targets

In practice:

- `spry serve` runs the pipeline and starts a dev/runtime process
- `spry build` runs the pipeline and emits deployable artifacts

Spry is intentionally explicit:

- the filesystem is the source of truth for route structure
- generated output is inspectable
- route behavior should not be hidden behind a large imperative DSL

## Important Repository Areas

### Public API

- `lib/spry.dart`
  main framework export surface
- `lib/app.dart`
  smaller app-oriented export surface
- `lib/config.dart`
  `defineSpryConfig(...)` and target enums
- `lib/builder.dart`
  builder-facing exports for config, scanning, and generation

### Runtime and Framework Internals

- `lib/src/app.dart`
  main `Spry` request handling pipeline
- `lib/src/routing.dart`
  route, middleware, and error router construction
- `lib/src/event.dart`
  request-scoped context passed to handlers
- `lib/src/errors.dart`
  `HTTPError` and framework error conversion
- `lib/src/public*.dart`
  static asset serving across runtimes

### Build Pipeline

- `lib/src/builder/config.dart`
  load and merge build config
- `lib/src/builder/scanner.dart`
  scan routes, middleware, errors, and hooks from the project tree
- `lib/src/builder/route_tree.dart`
  in-memory metadata model for scanned files
- `lib/src/builder/generator.dart`
  emit generated `app.dart`, `hooks.dart`, and `main.dart`
- `lib/src/builder/target_spec.dart`
  target-specific generation details

### CLI

- `bin/spry.dart`
  CLI entrypoint
- `bin/src/build.dart`
  `spry build`
- `bin/src/serve.dart`
  `spry serve`
- `bin/src/build_pipeline.dart`
  shared pipeline orchestration
- `bin/src/write.dart`
  write generated files and sync target assets

### Documentation and Site

- `README.md`
  package homepage and quick orientation
- `CHANGELOG.md`
  long-lived release history
- `sites/spry.medz.dev/`
  documentation source
- `sites/spry.medz.dev/snippets/`
  embedded docs snippets

### Examples

- `example/dart_vm/`
- `example/node/`
- `example/bun/`
- `example/cloudflare/`
- `example/vercel/`
- `example/netlify/`
- `example/knex_dart/`

These examples are useful smoke-test targets when runtime/build behavior changes.

## Routing Model

Spry uses filesystem routing.

Common mappings:

- `routes/index.dart` -> `/`
- `routes/about.get.dart` -> `GET /about`
- `routes/users/[id].dart` -> `/users/:id`
- `routes/[...slug].dart` -> `/**:slug`

Supported expressive segment syntax includes:

- embedded params
- regex params
- optional params
- repeated params
- single-segment wildcards

When changing route parsing behavior, always inspect:

- `lib/src/builder/scanner.dart`
- `test/scanner_test.dart`
- `test/generator_test.dart`
- docs in `sites/spry.medz.dev/guide/routing.md`

## HTTP and Runtime Model

Spry currently aligns with:

- `ht 0.3.x`
- `osrv 0.6.x`
- `roux 0.5.x`

Important current expectations:

- `Request` and `Response` use Fetch-style init objects
- manual remainder route strings must use `/**`, not `/*`
- `RequestInit` and `ResponseInit` are re-exported by Spry

When changing upstream alignment, verify:

- constructor examples in docs and README
- route syntax docs and examples
- `CHANGELOG.md`
- migration docs

## Typical Change Boundaries

### If you change public API

Also check:

- `lib/spry.dart`
- `lib/app.dart`
- examples
- docs snippets
- migration notes
- changelog entry

### If you change route scanning or generation

Also check:

- `scanner_test.dart`
- `generator_test.dart`
- docs routing examples
- generated output assumptions in CLI tests

### If you change CLI behavior

Also check:

- `bin/spry.dart`
- CLI tests
- help text
- README and docs command examples

### If you change runtime target output

Also check:

- `target_spec.dart`
- `build_pipeline.dart`
- target-specific examples
- deploy docs under `sites/spry.medz.dev/deploy/`

## Validation Checklist for Code Changes

Default validation commands:

```bash
dart analyze
dart test
```

When docs change:

```bash
cd sites/spry.medz.dev
npm run docs:build
```

When runtime/build behavior changes, smoke-test key examples:

```bash
cd example/dart_vm && dart pub get && dart run spry build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [medz/spry](https://github.com/medz/spry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
