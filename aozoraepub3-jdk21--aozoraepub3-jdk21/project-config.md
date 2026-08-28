---
trigger: always_on
description: These instructions tailor Copilot to this repository so it can generate correct, maintainable changes and avoid common pitfalls.
---

# AozoraEpub3 – Copilot Project Instructions

These instructions tailor Copilot to this repository so it can generate correct, maintainable changes and avoid common pitfalls.

## Project Overview
- Purpose: Convert Aozora-style text into EPUB 3.3 (backward compatible with EPUB 3.2), supporting Ruby, vertical writing, images, and device presets.
- Language/Build: Java 21 baseline (Java 25 compatible), Gradle 9.6.1, JUnit 4.13 tests.
- Templates: Apache Velocity templates under `template/` control most XHTML/CSS generation.
- CLI: `AozoraEpub3` (main class) orchestrates parsing, conversion, and packaging.

## Key Paths
- Java sources: `src/`
- Tests: `test/`
- Templates: `template/` (e.g., `template/OPS/css/vertical_text.vm`)
- Presets (INI): `presets/*.ini`
- Test data: `test_data/`
- Distribution scripts: `build/scripts/`

## Build & Run

### Important: Build Tasks (to avoid confusion)
This project uses custom build tasks:

1. **Build FAT JAR** (single runnable JAR)
  - Command: `./gradlew jar` (Windows: `gradlew.bat jar`)
  - Output: `build/libs/AozoraEpub3.jar`
  - Purpose: Single JAR including all dependencies

2. **Build distribution package** (ZIP/TAR)
  - Command: `./gradlew dist` (Windows: `gradlew.bat dist`)
  - Output:
     - `build/distributions/AozoraEpub3-<version>.zip`
     - `build/distributions/AozoraEpub3-<version>.tar.gz`
  - Contents: JAR + launcher scripts + docs + templates
  - Note: `distZip` is disabled. Use `dist`.

3. **Run tests**
  - Command: `./gradlew test` (Windows: `gradlew.bat test`)

### How to run
- Run CLI: `java -jar build/libs/AozoraEpub3.jar [options] input.txt`
- Sample conversion (UTF-8): `java -jar build/libs/AozoraEpub3.jar -of -d out input.txt`
- Launch GUI: run with no arguments `java -jar build/libs/AozoraEpub3.jar`

## CI & Validation
- GitHub Actions workflow builds, runs tests, generates sample EPUBs, and runs `epubcheck`.
- Basic checks for EPUB 3.3 and industry guide compliance are included as shell assertions.
- When adding tests, prefer small, deterministic unit tests over end-to-end unless necessary.

### IndexNow (docs delivery / search engine notification)
- When docs are updated, IndexNow submissions assume `docs/sitemap.xml` includes new/updated pages (the workflow auto-collects URLs from the sitemap).
- Host verification key: `docs/fad6fa3a81974f6aa0740a0861fbaefe.txt` (single-line key). Do not move this file when adding pages.
- Actions Variables: configure `INDEXNOW_HOST` and `INDEXNOW_BASE_URL` as needed. If unset, the workflow auto-detects from the first sitemap URL, but prioritize variable consistency when migrating hosts.
- Submission timing: on push (docs/** changes), manual runs (workflow_dispatch), and a daily schedule at 03:00 UTC (12:00 JST).
- In workflow logs, confirm “Discovered N URL(s)”, the first 20 samples, and that `urlList` in the request body contains all URLs. If not, review sitemap generation/published pages.

## Coding Guidelines
- Keep changes minimal and focused; align with existing style.
- Avoid introducing global state. If needed (e.g., Velocity), allow dependency injection.
- Favor small helpers over large monolith methods; keep public APIs stable.
- Validate inputs and fail fast with clear messages.
- Don’t add license headers unless explicitly requested.
- **Git Commits**: All commit messages **must be in Japanese**. Format should clearly describe the what/why of changes.

## Templates (Velocity) – Important
- Velocity resources live under `template/`. Use relative paths consistently from a configurable `templatePath`.
- Do NOT hard-code absolute paths. Tests and CI may run with different working directories.
- Avoid calling `Velocity.init()` unconditionally inside constructors. Prefer:
  - Accepting a `VelocityEngine` instance, or
  - Initializing only if not already configured.
- Keep placeholders and conditionals simple. Avoid mixing presentation with business logic.

## Presets / INI to CSS
- Device presets and INI values map to CSS via Velocity templates.
- Add or adjust CSS variables by threading values through the model/context used by templates.
- When adding new INI keys, update:
  - Parsing/validation logic
  - `Epub3Writer`/converter context population
  - Corresponding `.vm` templates
  - Unit tests covering both parsing and emitted CSS

## Testing
- Use JUnit 4 (`org.junit.Test`).
- Put fixtures under `test_data/` and avoid relying on the process working directory.
- Prefer unit tests for:
  - Parsing (INI, Aozora text features)
  - Small rendering functions
  - EPUB packaging helpers
- End-to-end tests calling the CLI can be flaky in Gradle workers due to path/resource differences. If unavoidable, run in a temporary directory and assert on produced `.epub` contents.
- Use `epubcheck` in CI for final validation; no network calls in tests.

## narou.rb Integration – Observed Behavior

This section records the **actual runtime settings** that narou.rb applies when calling AozoraEpub3, verified by comparing narou.rb-generated EPUBs against Java CLI output under controlled conditions (cross-validated via a .NET port of this tool that achieves byte-identical EPUB output for the same inputs).

### Settings narou.rb uses by default


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AozoraEpub3-JDK21/AozoraEpub3-JDK21](https://github.com/AozoraEpub3-JDK21/AozoraEpub3-JDK21) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
