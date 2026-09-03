---
trigger: always_on
description: This repository contains a privacy-first location-history visualizer. Users import a Google Maps Timeline JSON file and explore their journeys on an interactive map.
---

# AGENTS.md

## Project purpose

This repository contains a privacy-first location-history visualizer. Users import a Google Maps Timeline JSON file and explore their journeys on an interactive map.

Location history is extremely sensitive. Protecting it is a core system requirement.

## Product principles

1. Process imported files entirely in the browser.
2. Never transmit imported or derived location data.
3. Make privacy behavior explicit and verifiable.
4. Keep the initial workflow understandable without documentation.
5. Maintain an original visual identity.
6. Prefer correctness and clarity over adding more features.
7. Provide a fictional sample journey so users can evaluate the product safely.

## Working method

Before editing:

1. Inspect the repository structure and existing instructions.
2. Read the relevant implementation and tests.
3. Check the current working tree and preserve unrelated user changes.
4. Identify the smallest coherent change that satisfies the task.
5. State any assumption that materially affects behavior.

While editing:

- Keep changes scoped to the request.
- Follow existing conventions unless they conflict with this document.
- Do not perform broad rewrites without a concrete reason.
- Keep parsing, domain logic, map rendering, and UI state separated.
- Add or update tests alongside behavior changes.
- Use established dependencies before introducing new ones.
- Do not silently weaken privacy, accessibility, validation, or error handling.

After editing:

- Run formatting, linting, type checking, and relevant tests.
- Test the affected user flow.
- Review the final diff for unrelated changes.
- Report verification results and any known limitations.

## Technical baseline

Unless the repository already defines another stack, use:

- Next.js App Router
- React
- TypeScript in strict mode
- Tailwind CSS
- MapLibre GL JS
- Vitest
- Playwright

Do not migrate an established stack merely to match this baseline.

## Architecture

Keep these concerns independent:

- Source-format detection
- Timeline parsing
- Data normalization
- Geographic calculations
- Filtering
- Statistics
- Playback
- Map presentation
- User-interface state
- Export
- Privacy controls

Map components must consume normalized domain data. They must not understand every variation of Google’s source schemas.

New source formats should normally be implemented as adapters that produce the shared normalized model.

## Privacy and security rules

These rules are mandatory.

- Never upload imported files.
- Never send coordinates, timestamps, routes, filenames, or derived statistics to a server.
- Never include imported information in analytics, telemetry, crash reports, URLs, query parameters, or page titles.
- Never log raw imported records.
- Never store imported data in cookies, local storage, IndexedDB, caches, or service workers unless a user-facing requirement explicitly authorizes persistence.
- Do not use server actions, API routes, or remote parsing for imported data.
- Do not introduce third-party scripts without reviewing their privacy impact.
- Do not expose sensitive data through verbose errors.
- Do not use `eval` or execute content from imported files.
- Treat imported JSON as untrusted input.
- Validate coordinates, timestamps, array sizes, and nested structures.
- Prevent prototype-pollution patterns when transforming arbitrary objects.
- Remove in-memory dataset references when the user selects “Remove imported data.”
- Synthetic sample data must not resemble a real person’s precise location history.
- Map tiles will require ordinary network requests. Document this clearly and ensure imported coordinates are not deliberately added to tile-provider requests beyond normal tile selection behavior.

Any requested change that conflicts with these rules must be surfaced rather than implemented silently.

## Data handling

Use explicit TypeScript types for normalized data.

Parsers should:

- Detect their supported schema
- Validate required fields
- Tolerate optional-field variations
- Skip recoverable invalid records
- Collect structured warnings
- Fail clearly when no usable timeline remains
- Return normalized values
- Avoid mutating the source object

Clear or release raw-file references once normalization completes.

For large files, prefer streaming techniques or Web Workers where practical. Do not freeze the main interface during expensive parsing.

## Geographic correctness

- Validate latitude as `-90...90`.
- Validate longitude as `-180...180`.
- Use a documented geodesic distance method.
- Keep units explicit.
- Handle timestamps and time zones deliberately.
- Do not silently interpret missing time-zone information as local time unless this is documented.
- Treat implausible jumps as possible measurement errors rather than silently deleting them.
- Preserve enough information to explain ignored or repaired records.

## Map implementation

- Use MapLibre GeoJSON sources and layers for routes.
- Do not create thousands of DOM markers.
- Keep source updates controlled and measurable.
- Fit bounds only when appropriate; do not constantly override the user’s viewport.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blegoh/Location-Timeline](https://github.com/blegoh/Location-Timeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
