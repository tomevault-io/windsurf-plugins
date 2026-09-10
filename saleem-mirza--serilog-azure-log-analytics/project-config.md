---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Serilog sink that ships log events to Azure Monitor through the Logs Ingestion API (Data Collection Rules), published as the `Serilog.Sinks.AzureLogAnalytics` NuGet package. The library lives in `src/`, its tests in `test/Serilog.Sinks.AzureLogAnalytics.Tests/`. There is no solution file, so build and test commands take a directory or a project path.

## Commands

All commands run from `src/`:

```bash
dotnet restore
dotnet build --no-restore
dotnet pack                       # produces the NuGet package
dotnet build -f net8.0            # single target framework
```

Tests run from `test/Serilog.Sinks.AzureLogAnalytics.Tests/`:

```bash
dotnet test
dotnet test --filter RejectedBatchIsRetried    # one test
```

There is no lint or format step.

The tests drive the sink through the public `WriteTo.AzureLogAnalytics(...)` API against an `HttpListener` on a loopback port (`Collector.cs`), with a stub `TokenCredential`, so nothing touches Azure or the network. They assert on what reaches the wire: batching, request URL, bearer header, envelope shape, and the retry path. Adding a case means adding a `[Fact]`, not a mock.

The suite takes about 30 seconds, and that is inherent rather than slack. Serilog's `BufferingTimeLimit` (10 seconds here) governs when a partial batch flushes, and its retry scheduler paces reattempts at roughly 10 seconds. Both are wall-clock waits with no seam to shorten them. Keep new tests inside the existing waits where possible.

The test project targets `net8.0` to match the library and CI, with `RollForward` set to `Major` so the test host runs on a newer runtime when 8.0 is not installed.

Targets `netstandard2.0;net8.0` with `LangVersion` 8.0. Language features newer than C# 8 will not compile. The assembly is strong-named with `src/Serilog.snk`, so a build needs that key file present.

CI (`.github/workflows/dotnet.yml`) runs restore, build, test, and pack on push and PR against `master`, which is the release branch. The default working branch is `dev`, so CI does not fire for it: run `dotnet test` yourself before pushing to `dev`.

## Architecture

Two types carry the sink, plus a converter:

1. `Sinks/AzureLogAnalytics/AzureLogAnalyticsSink.cs` implements `Serilog.Core.IBatchedLogEventSink`. Serilog's own `BatchingSink` owns all queuing, batching, retry, and shutdown flushing, so this class holds no threads or buffers. `EmitBatchAsync` wraps each `LogEvent` into `{ TimeGenerated, Event, Message }` and POSTs the array to `{Endpoint}/dataCollectionRules/{ImmutableId}/streams/{StreamName}?api-version=2023-01-01`. `OnEmptyBatchAsync` returns a completed task. Auth resolves once, on the first batch, to a single `TokenCredential` held in a `Lazy<TokenCredential>`: `LoggerCredential.TokenCredential` when the caller supplies one, otherwise an `Azure.Identity` `ClientSecretCredential` built from tenant/client/secret. The deferral matters because `ClientSecretCredential` validates tenant/client/secret in its own constructor, and the sink constructor runs inside `CreateLogger()`: building it eagerly turns a bad secret into a startup crash instead of a failed batch that Serilog retries and reports through `SelfLog`. Each batch calls `GetTokenAsync` and sets `Authorization` on its own `HttpRequestMessage`, so the static shared `httpClient` carries no per-sink state. There is no local token cache, because `TokenCredential` implementations cache and refresh internally.

2. `LoggerConfigurationExtensions.cs` exposes the two `WriteTo.AzureLogAnalytics(...)` overloads, differing by a leading `ITextFormatter` parameter. The shorter one delegates to the longer with a null formatter, so the sink is constructed in one place. It translates `ConfigurationSettings` into Serilog's `BatchingOptions`: `BatchSize` maps to `BatchSizeLimit`, `BufferSize` to `QueueLimit`, and `BufferingTimeLimit` (10 seconds) plus `RetryTimeLimit` (2 minutes) are set explicitly to preserve the cadence of the hand-rolled batcher that Serilog's batching replaced. `ConfigurationSettings` and `LoggerCredential` property names must stay in sync with the `appsettings.json` keys documented in README.md, since `Serilog.Settings.Configuration` binds by name.

Event serialization goes through `LoggerJsonConverter`, a `JsonConverter<LogEvent>` that delegates to the supplied `ITextFormatter` (defaulting to Serilog's `JsonFormatter`) and writes the result with `WriteRawValue`. It decides the payload of the `Event` property only. The surrounding envelope and the serializer options (naming policy, `ReferenceHandler.IgnoreCycles`) are set in `AzureLogAnalyticsSink`.

The envelope is built as `Dictionary<string, object>`, and System.Text.Json does not apply `PropertyNamingPolicy` to dictionary keys. `TimeGenerated`, `Event`, and `Message` therefore stay PascalCase on the wire even under `NamingStrategy.CamelCase`, which matches the DCR column names. Switching the envelope to an anonymous type or a POCO would rename those columns and break ingestion.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saleem-mirza/serilog-azure-log-analytics](https://github.com/saleem-mirza/serilog-azure-log-analytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
