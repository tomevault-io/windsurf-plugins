---
trigger: always_on
description: Shared instructions for contributors and coding agents working in this repository.
---

# AGENTS.md

Shared instructions for contributors and coding agents working in this repository.

## Project Overview

PaliPractice is a cross-platform language learning app for practicing Pali noun declensions and verb conjugations. Built with .NET 10 and Uno Platform for Windows, Mac, Linux, Android, and iOS.

Uno Platform implements the entire WinUI and WinRT API (like Microsoft.UI) surface across platforms. So when developing in Uno, think as an experienced WinUI/WinRT developer.

## Key Commands

### Build and Run
```bash
# Build for desktop (Windows/Mac/Linux) - target project directly to avoid test project issues
cd PaliPractice
dotnet build PaliPractice/PaliPractice.csproj -f net10.0-desktop

# Run on desktop
dotnet run --project PaliPractice/PaliPractice.csproj -f net10.0-desktop

# Build for other platforms
dotnet build PaliPractice/PaliPractice.csproj -f net10.0-ios
dotnet build PaliPractice/PaliPractice.csproj -f net10.0-android

# Run tests (uses net10.0, not platform-specific)
dotnet test PaliPractice.Tests/PaliPractice.Tests.csproj
```

### Database Generation

Run from the repository root. See [scripts/SETUP.md](scripts/SETUP.md) for
pinned input acquisition; ordinary app builds use the bundled database.

```bash
# Build an isolated English candidate from pinned inputs (see scripts/SETUP.md)
.venv/bin/python scripts/extract_nouns_and_verbs.py build --manifest <inputs.json> --output <new-candidate-directory>

# Validate the candidate's current structural contract
.venv/bin/python scripts/extract_nouns_and_verbs.py validate <candidate-directory>

# Full database rebuild (if needed)
cd dpd-db
uv run scripts/build/db_rebuild_from_tsv.py
uv run python db/inflections/create_inflection_templates.py
uv run python db/inflections/generate_inflection_tables.py
```

## Architecture Overview

### Technology Stack
- **Framework**: Uno Platform 6 with .NET 10
- **UI Pattern**: MVVM with C# Markup (fluent API)
- **Database**: SQLite via sqlite-net-base and SQLitePCLRaw.bundle_e_sqlite3
- **Data Source**: Digital Pāḷi Dictionary (DPD) as git submodule

### Project Structure
- `/PaliPractice/PaliPractice/` - Main app code
  - `Models/` - Lemma, noun/verb, detail, and inflection models
  - `Presentation/` - Pages and ViewModels
  - `Services/` - DatabaseService for SQLite access
  - `Data/pali.db` - Bundled dictionary; version and provenance sidecars are adjacent
  - `Platforms/` - Platform-specific implementations

- `/scripts/` - Python extraction pipeline
  - `extract_nouns_and_verbs.py` - Main extraction script
  - `SETUP.md` - Comprehensive setup documentation
  
- `/dpd-db/` - DPD submodule with dictionary data

### Database Schema
The bundled dictionary uses these tables:
- **nouns / verbs**: DPD headwords, stable lemma IDs, EBT counts, stems/paradigms, and explicit practice-primary selection
- **nouns_details / verbs_details**: English meanings and language-neutral examples
- **localized_meanings**: Russian/Spanish meanings keyed by DPD headword and language
- **nouns_corpus_forms / verbs_corpus_forms**: Attested `(headword_id, form_id)` pairs
- **nouns_irregular_forms / verbs_irregular_forms**: Scoped form IDs plus spellings needed for reconstruction

Regular endings live in C# grammar tables. Full corpus spellings and primary-form
contracts live in `scripts/generated` as build/test evidence, not app assets.
User mastery, settings, and history live separately in `practice.db`.

### Key Implementation Details

1. **Data Selection**: Uses EBT frequency to select 1,500 noun lemmas and 750 verb lemmas; practice paradigms follow the append-only identity registry
2. **Navigation**: Route-based navigation with Shell pattern
3. **Database Access**: SQLite access through DatabaseService and repositories
4. **UI Construction**: C# Markup fluent API instead of XAML

### State Management
- **Minimize mutable state**: Only introduce new state when absolutely necessary; prefer derived/computed values over stored state
- **Avoid state duplication**: Maintain a single source of truth; never store the same information in multiple places
- **Prefer enums over constants**: Use enums for finite sets of related values instead of multiple booleans or string/int constants
- **Favor pure functions**: Pass data as explicit parameters rather than relying on instance state; this improves testability and reduces side effects
- **Reuse existing state**: Before adding a new boolean or flag, check if existing state can express the same condition
- **Avoid state explosion**: Multiple independent booleans create exponential state combinations; consolidate into enums or state objects when states are mutually exclusive

### Working with the Codebase

When modifying code:
- Follow existing C# Markup patterns in Presentation layer
- Maintain MVVM separation (ViewModels handle logic)
- Database models are in Models/ directory
- Platform-specific code goes in Platforms/ subdirectories
- `pali.db` is a packaged asset. Platforms may read it directly or copy it into app storage; the existing version check replaces older copies.
- `history-v1.1.json.gz` is an immutable embedded resource for legacy history backfill, loaded only during migration when needed.

**UI Text Guidelines:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DhammaBytes/PaliPractice](https://github.com/DhammaBytes/PaliPractice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
