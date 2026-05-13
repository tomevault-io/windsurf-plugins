---
trigger: always_on
description: <!-- Copyright © 2025-26 l5yth & contributors -->
---

<!-- Copyright © 2025-26 l5yth & contributors -->
<!-- Licensed under the Apache License, Version 2.0 (see LICENSE) -->

# Repository Guidelines

Keep code as modular as possible to reduce duplication and improve reusability and readability — this applies to tests as well as production code. If a module grows large, split it into a submodule structure. Prefer composing small, single-purpose units over monolithic files.

Make sure all tests pass for Python (`pytest`), Ruby (`rspec`), and JavaScript (`npm test`).

All code must be 100% unit tested — every line, branch, and code path must have a unit test. "100%" is the floor, not the ceiling: smoke tests, integration tests, and end-to-end tests come on top of that. No new code ships without matching unit tests.

All code must be 100% documented according to the language's API-doc standard (PDoc for Python, RDoc for Ruby, JSDoc for JavaScript, rustdoc for Rust, dartdoc for Dart). Documentation must be sufficient to generate complete API docs from source. In addition to API-level docs, add inline comments wherever the logic is not immediately self-evident.

Every file in the repository must carry an Apache v2 license notice using the exact string `Copyright © 2025-26 l5yth & contributors`. **Source-code files** (`.rb`, `.py`, `.js`, `.rs`, `.dart`, etc.) must include the full Apache v2 license header block. **Non-source files** (docs, configs, YAML, TOML, Dockerfiles, etc.) must include a short 2-line Apache v2 notice (copyright line + license reference).

Run linters for Python (`black`) and Ruby (`rufo`) to ensure consistent code formatting.

## Project Structure & Module Organization
The repository splits runtime and ingestion logic. `web/` holds the Sinatra dashboard (Ruby code in `lib/potato_mesh`, views in `views/`, static bundles in `public/`).

`data/` hosts the Python Meshtastic ingestor plus migrations and CLI scripts. The ingestor is structured as the `data/mesh_ingestor/` package with the following key modules: `daemon.py` (main loop), `handlers.py` (packet processing), `interfaces.py` (interface helpers), `config.py` (env-driven config), `events.py` (TypedDict event schemas), `mesh_protocol.py` (MeshProtocol base), `node_identity.py` (canonical node ID utilities), `decode_payload.py` (CLI protobuf decoder), and the `protocols/` subpackage (currently `meshtastic.py`). API contracts for all POST ingest routes are documented in `data/mesh_ingestor/CONTRACTS.md`. API fixtures and end-to-end harnesses live in `tests/`. Dockerfiles and compose files support containerized workflows.

`matrix/` contains the Rust Matrix bridge; build with `cargo build --release` or `docker build -f matrix/Dockerfile .`, and keep bridge config under `matrix/Config.toml` when running locally.

## Build, Test, and Development Commands
Run dependency installs inside `web/`: `bundle install` for gems and `npm ci` for JavaScript tooling. Start the app with `cd web && API_TOKEN=dev ./app.sh` for local work or `bundle exec rackup -p 41447` when integrating elsewhere.

Prep ingestion with `python -m venv .venv && pip install -r data/requirements.txt`; `./data/mesh.sh` streams from live radios. `docker-compose -f docker-compose.dev.yml up` brings up the full stack.

Container images publish via `.github/workflows/docker.yml` as `potato-mesh-{service}-linux-$arch` (`web`, `ingestor`, `matrix-bridge`), using the Dockerfiles in `web/`, `data/`, and `matrix/`.

## Coding Style & Naming Conventions
Use two-space indentation for Ruby and keep `# frozen_string_literal: true` at the top of new files. Keep Ruby classes/modules in `CamelCase`, filenames in `snake_case.rb`, and feature specs in `*_spec.rb`.

JavaScript follows ES modules under `public/assets/js`; co-locate components with `__tests__` folders and use kebab-case filenames. Format Ruby via `bundle exec rufo .` and Python via `black`. Skip committing generated coverage artifacts.

## Flutter Mobile App (`app/`)
The Flutter client lives in `app/`. Keep only the mobile targets (`android/`, `ios/`) under version control unless you explicitly support other platforms. Do not commit Flutter build outputs or editor cruft (`.dart_tool/`, `.flutter-plugins-dependencies`, `.idea/`, `.metadata`, `*.iml`, `.fvmrc` if unused).

Install dependencies with `cd app && flutter pub get`; format with `dart format .` and lint via `flutter analyze`. Run tests with `cd app && flutter test` and keep widget/unit coverage high—no new code without tests. Commit `pubspec.lock` and analysis options so toolchains stay consistent.

## Testing Guidelines
Ruby specs run with `cd web && bundle exec rspec`, producing SimpleCov output in `coverage/`. Front-end behaviour is verified through Node’s test runner: `cd web && npm test` writes V8 coverage and JUnit XML under `reports/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [l5yth/potato-mesh](https://github.com/l5yth/potato-mesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
