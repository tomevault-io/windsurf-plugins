---
trigger: always_on
description: This file is a solution-wide navigation guide for humans and coding agents working in `NINA.sln`. It complements the per-project `ARCHITECTURE.md` files and stays focused on boundaries that are backed up by the code.
---

# AGENTS.md

This file is a solution-wide navigation guide for humans and coding agents working in `NINA.sln`. It complements the per-project `ARCHITECTURE.md` files and stays focused on boundaries that are backed up by the code.

For repository-wide contribution workflow, branch expectations, release-note updates, and general coding rules, also read [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Scope

This guide covers the projects listed in `NINA.sln`.

## Documentation Boundary

- `NINA.Docs` is a git submodule declared in `.gitmodules` and points to `https://github.com/isbeorn/nina.docs.git`.
- It is not part of the `NINA.sln` project architecture documented below.
- If a code change also requires user-facing documentation updates, handle that separately in the `NINA.Docs` submodule / `nina.docs` repository, following the documentation notes in [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Repository Knowledge

- Treat repository-local, versioned files as the system of record for future contributors and coding agents.
- Keep this file as a map. Put detailed subsystem guidance in the owning `ARCHITECTURE.md`, `CONTRIBUTING.md`, or a focused checked-in reference.
- If a review, bug, or repeated agent mistake reveals a durable rule, update the nearest doc, test, analyzer, script, or route map instead of relying on off-repo memory.

## Code Style And Formatting

- Maintain the existing line-ending style of every touched file; default to CRLF for new files unless the target location dictates LF.
- Treat the root [`.editorconfig`](.editorconfig) as the canonical C# style source. It covers indentation, line endings, namespace style, `using` placement, `var` preferences, naming, and selected analyzer severities.
- Do not introduce new warnings casually. If an existing warning blocks a focused cleanup, keep the fix scoped and document any intentional deferral in the PR.
- For XAML, follow surrounding file style; no repo-wide XAML formatter configuration is checked in.
- Prefer modern C# supported by the target project. For new or refactored MVVM code, prefer `CommunityToolkit.Mvvm` where it fits instead of expanding legacy relay-command patterns.

## Project Architecture Docs

Read the project-local architecture doc before making non-trivial changes in that project:

- [`NINA/ARCHITECTURE.md`](NINA/ARCHITECTURE.md)
- [`NINA.Astrometry/ARCHITECTURE.md`](NINA.Astrometry/ARCHITECTURE.md)
- [`NINA.Core/ARCHITECTURE.md`](NINA.Core/ARCHITECTURE.md)
- [`NINA.CustomControlLibrary/ARCHITECTURE.md`](NINA.CustomControlLibrary/ARCHITECTURE.md)
- [`NINA.Equipment/ARCHITECTURE.md`](NINA.Equipment/ARCHITECTURE.md)
- [`NINA.Image/ARCHITECTURE.md`](NINA.Image/ARCHITECTURE.md)
- [`NINA.MGEN/ARCHITECTURE.md`](NINA.MGEN/ARCHITECTURE.md)
- [`NINA.Platesolving/ARCHITECTURE.md`](NINA.Platesolving/ARCHITECTURE.md)
- [`NINA.Plugin/ARCHITECTURE.md`](NINA.Plugin/ARCHITECTURE.md)
- [`NINA.Profile/ARCHITECTURE.md`](NINA.Profile/ARCHITECTURE.md)
- [`NINA.Sequencer/ARCHITECTURE.md`](NINA.Sequencer/ARCHITECTURE.md)
- [`NINA.Sequencer.Generators/ARCHITECTURE.md`](NINA.Sequencer.Generators/ARCHITECTURE.md)
- [`NINA.Setup/ARCHITECTURE.md`](NINA.Setup/ARCHITECTURE.md)
- [`NINA.SetupBundle/ARCHITECTURE.md`](NINA.SetupBundle/ARCHITECTURE.md)
- [`NINA.Test/ARCHITECTURE.md`](NINA.Test/ARCHITECTURE.md)
- [`NINA.WPF.Base/ARCHITECTURE.md`](NINA.WPF.Base/ARCHITECTURE.md)

Note: the solution project is named `NINA.PlateSolving`, but the folder on disk is `NINA.Platesolving`.

## Solution Map

The solution has a clear layering pattern.

### Foundation

- `NINA.Core`
  Shared utilities, logging, localization, enums, common models, SQLite EF context, and generated protobuf contracts.
- `NINA.Profile`
  Persisted user profile and typed settings model.
- `NINA.Astrometry`
  Astronomy math, coordinate types, night/twilight calculations, catalog queries.

### Runtime Domain Libraries

- `NINA.Image`
  Image model, file formats, analysis, star detection, rendering helpers.
- `NINA.MGEN`
  Standalone MGEN2/MGEN3 transport/protocol library.
- `NINA.Equipment`
  Device abstractions and concrete ASCOM/Alpaca/native adapters.
- `NINA.Platesolving`
  Plate-solver integrations and orchestration.

### Shared UI Infrastructure

- `NINA.CustomControlLibrary`
  Reusable custom WPF controls and themes.
- `NINA.WPF.Base`
  Shared mediators, dockable/view-model base classes, equipment UI support, sky survey subsystem.

### Extensibility And Sequencing

- `NINA.Plugin`
  Plugin manifests, loading, installation, compatibility checks, MEF composition.
- `NINA.Sequencer`
  Advanced sequencer engine, entity model, serialization, target/template storage, expressions/symbols.
- `NINA.Sequencer.Generators`
  Roslyn source generator used by sequencer expression-backed properties.

### App Shell, Packaging, Verification

- `NINA`
  Executable WPF app, DI composition root, main shell, app-specific views/view models, runtime assets.
- `NINA.Setup`
  WiX MSI project.
- `NINA.SetupBundle`
  WiX Burn bootstrapper.
- `NINA.Test`
  NUnit test suite.

### Non-`NINA.*` Solution Projects


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isbeorn/nina](https://github.com/isbeorn/nina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
