---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jiten is a free, open-source platform for Japanese immersion learners that analyses Japanese media to provide detailed statistics (character count, difficulty, vocabulary lists, frequency lists, etc.) and generates Anki decks. See https://jiten.moe for the live platform.

## Repository Structure

```
/
├── Jiten.Api/            # ASP.NET Core Web API
├── Jiten.Cli/            # Command-line tool for batch processing
├── Jiten.Core/           # Core library with domain models and data access
├── Jiten.Parser/         # Japanese text parsing engine
├── Jiten.Tests/          # xUnit test suite
├── Jiten.Web/            # Nuxt 4 frontend application (Vue 4, PrimeVue 4, TailwindCSS 4)
├── Shared/               # Shared resources (dictionaries, ML models, config)
└── Jiten.sln             # Main .NET solution file
```

## Build & Run Commands

```bash
# Backend (.NET 9.0) - run from root directory
dotnet build Jiten.sln
dotnet run --project Jiten.Api/Jiten.Api.csproj          # API at https://localhost:7299
dotnet run --project Jiten.Cli/Jiten.Cli.csproj -- [options]
dotnet test Jiten.Tests/Jiten.Tests.csproj
dotnet test --filter "FullyQualifiedName~DeconjugatorTests.DeconjugationTest"  # Single test

# Frontend - run from Jiten.Web/
pnpm install && pnpm dev    # Dev server at https://localhost:3000
pnpm build                  # Production build
pnpm lint / pnpm lintfix    # Lint
```

## Solution Architecture

**Jiten.Core** - Domain models (`Deck`, `DeckWord`, `JmDictWord`, `ExampleSentence`, etc.), data access (`JitenDbContext` for main data, `UserDbContext` for auth), PostgreSQL + EF Core, metadata providers (Anilist, VNDB, Google Books, IGDB, TMDB).

**Jiten.Parser** - Japanese text parsing engine. MorphologicalAnalyser (Sudachi native interop), Deconjugator (rule-based from `deconjugator.json`), Parser (JMDict lookup pipeline), Redis-backed caching. See [Processing Pipeline](#japanese-text-processing-pipeline) below.

**Jiten.Cli** - Batch processing CLI. Format-specific extractors (KiriKiri, BGI, YuRis, PSB, etc.), bruteforce regex extractor, text/subtitle/manga extractors. Commands for parsing, extraction, metadata download, dictionary import.

**Jiten.Api** - ASP.NET Core Web API. JWT + API Key + Google OAuth authentication. Hangfire background jobs (`ParseJob`, `ComputationJob`, `FetchMetadataJob`, `ReparseJob`). Rate limiting with tiered access. OpenTelemetry observability. Swagger at `/swagger`.

**Jiten.Tests** - xUnit + FluentAssertions. Unit tests: `DeconjugatorTests`, `MorphologicalAnalyserTests`, `FormSelectionTests`, `FsrsTests`. Integration tests under `Integration/` use `WebApplicationFactory` with SQLite in-memory (see [Integration Testing](#integration-testing)).

**Jiten.Web** - Nuxt 4 frontend (Vue 4, TypeScript, Pinia). PrimeVue 4 components with TailwindCSS 4. API calls via `useApiFetch` composable with JWT auto-refresh. File-based routing with `auth`/`authAdmin` middleware. Stores: `authStore` (JWT + Google OAuth), `jitenStore` (preferences), `displayStyleStore` (UI). Nuxt auto-imports composables, components, and utils.

### Dependency Flow
- Jiten.Api → Jiten.Core, Jiten.Parser
- Jiten.Cli → Jiten.Core, Jiten.Parser
- Jiten.Parser → Jiten.Core
- Jiten.Tests → Jiten.Parser

## Database Architecture

**JitenDbContext (schema: jiten)** - `Decks` (media entries with stats, parent-child relationships, external links), `DeckWords` (WordId + ReadingIndex + Occurrences), `DeckRawText`, `ExampleSentences`.

**JitenDbContext (schema: jmdict)** - `Words` (JMDict entries with Readings[], PartsOfSpeech[], PitchAccents[]), `Definitions` (multilingual), `Lookups` (text → WordIds), `WordFrequencies`.

**UserDbContext** - ASP.NET Identity, `UserCoverage`, `UserMetadata`, `ApiKeys`, `RefreshTokens`, `UserKnownWord`, `UserDeckPreference`, `UserWordSetState`.

**Key indexes**: PGroonga full-text search on Decks titles, WordId + ReadingIndex composites, DeckId indexes.

### EF Migrations
```bash
dotnet ef database update --project Jiten.Core --startup-project Jiten.Core
dotnet ef migrations add MigrationName --project Jiten.Core --startup-project Jiten.Core --context JitenDbContext
```

## Japanese Text Processing Pipeline

1. **Morphological Analysis** (`Jiten.Parser/MorphologicalAnalyser.cs`): Sudachi tokenises text → WordInfo objects (Text, DictionaryForm, PartOfSpeech, NormalizedForm)
2. **Deconjugation** (`Jiten.Parser/Deconjugator.cs`): Applies rules from `deconjugator.json` → possible base forms with conjugation history
3. **JMDict Lookup** (`Jiten.Parser/Parser.cs`): Queries Lookups table → matches by POS compatibility → priority scoring for ambiguous matches
4. **Result** (`Jiten.Core/Data/DeckWord.cs`): WordId, ReadingIndex, OriginalText, Conjugations, Occurrences

**Caching**: Redis-backed word cache by (Text, POS, DictionaryForm) tuples. JMDict cache populated in 10K batches. Cache failures are non-fatal. **You MUST flush Redis after parser changes** (`dotnet run --project Jiten.Cli -- --flush-redis`).

## Autonomous Parser Testing

**Test commands:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sirush/Jiten](https://github.com/Sirush/Jiten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
