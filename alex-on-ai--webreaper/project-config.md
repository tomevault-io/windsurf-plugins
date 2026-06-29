---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

WebReaper — declarative, parallel web scraper/crawler library for .NET, published as the `WebReaper` NuGet package. Library lives in `WebReaper/`; `Examples/` and `Misc/` are consumers, not packaged.

## Commands

```bash
dotnet build WebReaper.sln                  # build all
dotnet test                                  # all tests (xUnit)
dotnet test WebReaper.Tests/WebReaper.UnitTests/WebReaper.UnitTests.csproj   # unit only (fast, offline)
dotnet test --filter "FullyQualifiedName~SimpleTest"   # single test by name
dotnet run --project Examples/WebReaper.ConsoleApplication   # run an example scraper
```

### Test reality

- **Unit tests** (`WebReaper.Tests/WebReaper.UnitTests`) — offline, parse `TestData/TestPage.html`. Use these for fast iteration.
- **Integration tests** (`WebReaper.Tests/WebReaper.IntegrationTests`) — hit the **live** site `alexpavlov.dev`, launch real Puppeteer/Chromium, and assert after fixed `Task.Delay` (up to 25s). Slow and network-flaky by design; failures there are often environmental, not regressions.

### Toolchain

All projects target **net10.0**; `global.json` pins SDK `10.0.100` (`rollForward: latestMinor`, no prerelease) and `.github/workflows/CI.yml` installs `10.0.x` — aligned. Build with a .NET 10 SDK. CI runs `dotnet restore` → `dotnet build` → unit tests over the **whole solution** (Examples/ and Misc/ included), so verify with a full-solution build, not just the library project.

## Architecture

Fluent builder → immutable config + pluggable spider → parallel job loop.

**Build path (ADR-0025, widened by ADR-0040 + ADR-0067):** a scrape begins with a **Crawl seed**. `ScraperEngineBuilder.Crawl(urls)` / `.CrawlWithBrowser(urls)` are **static** → return `ICrawlSeed`; the seed has four strategy terminals: `.Extract(schema)` (Schema-driven JSON via the deterministic fold), `.AsMarkdown()` (no-schema LLM-ready Markdown via `MarkdownContentExtractor`, ADR-0040), `.ExtractInferred(goal?)` (runtime schema inference via `LearnedSchemaContentExtractor` + `ISchemaInferrer`, ADR-0067), or `.ExtractWithPrompt(chatClient, instruction)` (schema-free LLM extraction via `PromptContentExtractor`, ADR-0084), all yielding the `ScraperEngineBuilder`. The builder's constructor is `internal` (test-only via `InternalsVisibleTo`), so it is unreachable without the seed: "build with no start URLs or no extraction strategy" is structurally impossible, not a runtime throw. It composes two internal builders:
- `ConfigBuilder` (internal) → immutable `ScraperConfig` (start URLs, the `LinkPathSelector` chain, crawl limit, headless flag).
- `SpiderBuilder` (internal) → runtime components (loaders, parsers, sinks, trackers, cookie storage), all with in-memory defaults.

Terminate with `BuildAsync()` (persists the config to `IScraperConfigStorage`, constructs a `Spider`, returns a `ScraperEngine`) or `Build()` (just the `ScraperConfig`, for a distributed start endpoint to persist). The ADR-0009 distributed-worker reduced shell is a **separate public seam**: `DistributedSpiderBuilder` (seedless, no `BuildAsync`; `.BuildSpider()` → `ISpider`) — "two seams, not one bug".

**Run path (ADR-0022):** `ScraperEngine` *is* the in-process **Crawl driver**. `RunAsync` seeds the `IScheduler` with one `Job` per start URL, then drives `Parallel.ForEachAsync` over `Scheduler.GetAllAsync()` (an async stream). Each job → `Spider.CrawlAsync` (wrapped in the **Retry policy** — `IRetryPolicy`, ADR-0026; the default `FixedAttemptsRetryPolicy` runs four attempts and never retries `OperationCanceledException`) returns a closed `JobReport`; the driver interprets it — applies the visited-link idempotency authority (atomic test-and-set), runs `ParsedData` through the page-processor pipeline (`IPageProcessor`, ADR-0038) then fans the surviving record to the sinks, enqueues child jobs, drives the **Outstanding-work latch**. The **Stop rule** verdict is a *value*, never a thrown exception (`PageCrawlLimitException` was removed in 8.0.0); on a concluded crawl the driver ends its own consumption of the job stream — `IScheduler.Complete()` was removed (ADR-0037), since durable schedulers could not honour it. Authoritative: `docs/adr/0022-crawl-driver-and-outstanding-work-latch.md`, `docs/adr/0026-retry-policy-seam.md`, `docs/adr/0037-stop-ceases-consumption.md`.

**Job model:** `Job` is a record carrying the URL, an `ImmutableQueue<LinkPathSelector>`, parent backlinks, and `PageType` (Static vs Dynamic). The crawl-vs-parse decision is **not** stored on `Job` (`Job.PageCategory` was removed) — it is computed by `CrawlStep` from the selector chain (0 left ⇒ parse the target page; exactly 1 with pagination ⇒ paginate; else follow links) and returned as a closed `CrawlOutcome` sum (`Parsed | Followed | Paginated`). Chain length is the state machine: each step dequeues one selector. Authoritative: `CONTEXT.md` + `docs/adr/0001-crawl-outcome-closed-sum.md`.

**`Spider.CrawlAsync` per job (ADR-0022 — the Spider only *reports*):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alex-on-ai/WebReaper](https://github.com/alex-on-ai/WebReaper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
