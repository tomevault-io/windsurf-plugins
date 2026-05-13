---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow Rules

- **Always execute immediately** — never stop at planning. Make file changes first, then describe what was done. If asked to implement something, write the code now, don't outline steps.
- **Always confirm completion explicitly** — after finishing work, state: what files were changed, build status, and test results.
- **Always write tests** — any bug fix or new feature must include unit tests. Do not wait to be asked. Run them and confirm they pass.
- **Always verify the build** — run `dotnet build` after changes. If errors exist, fix them and rebuild. Do not report completion until the build is clean.
- **Always add XML doc comments** — every new file must have `///` XML doc comments on all public types and members. Follow the Documentation Standards section below.

## Testing

- Tests live in `./tests/` split into three categories — see `tests/README.md` for the full picker:
  - **`tests/unit/`** — `*.UnitTests` projects. Pure NSubstitute mocks, no I/O, no `Task.Delay > 50ms`, no containers, no HTTP. Total wall-clock target: **< 60 s**. Runs on every PR and push.
  - **`tests/integration/`** — `*.IntegrationTests` projects. Hits real DB via Testcontainers, `WebApplicationFactory`, Hangfire in-memory, etc. Runs on every PR + push.
  - **`tests/regression/FlowOrchestrator.RegressionTests/`** — timing-sensitive (cron / polling / timeout) and concurrency stress (64-task gate contests). Runs nightly + on push to `main` + manual dispatch. Has its own `xunit.runner.json` with parallelization disabled.
- Framework: **xUnit + NSubstitute**. Use plain xUnit `Assert.*` — do **not** add FluentAssertions, Shouldly, or any fluent-assertion library.
- **AAA pattern** is mandatory: every `[Fact]`/`[Theory]` body must contain three comment blocks, in order — `// Arrange`, `// Act`, `// Assert`. Shared fixture setup in fields/constructor is allowed and does not count as the body's Arrange (a body block may be empty if there is genuinely nothing to do).
- **Solution filters** — pick the right one for what you're running:
  - `dotnet test FlowOrchestrator.UnitTests.slnf` — fast feedback (~30 s).
  - `dotnet test FlowOrchestrator.IntegrationTests.slnf` — needs Docker.
  - `dotnet test FlowOrchestrator.RegressionTests.slnf` — slow, run before merging anything that touches scheduling, polling, or concurrency primitives.
- **Anti-flakiness rules** (mandatory for any test you write):
  - Never assert an *upper bound* on `Stopwatch.Elapsed` — that pattern is the classic source of CI flakiness.
  - Never poll on a counter with a wall-clock deadline (`while (counter == 0) Task.Delay(...)`). Wait on a logical event — `TaskCompletionSource` set by the handler / store / dispatcher.
  - When you need a real sleep budget (timeout assertions), pick a generous one (≥ 30 s) so CI CPU contention doesn't trip it.
- New tests go in the matching project, picked by category first: a pure unit test for `FlowOrchestrator.Core` lives in `tests/unit/FlowOrchestrator.Core.UnitTests`; an integration test for the same component lives in `tests/integration/FlowOrchestrator.{Component}.IntegrationTests`.

## Build & Verification

- Before reporting a task complete: `dotnet build` must show 0 errors, 0 warnings (or document why warnings are acceptable).
- If build fails after a fix, keep iterating — do not stop and ask unless truly stuck after 3+ attempts.

### HTTP endpoints — small things AI keeps forgetting

When adding ANY HTTP endpoint that returns >1 KB typical payload (HTML page, JSON API, file download, etc.), verify the following before declaring the task complete:

1. **Honor `Accept-Encoding`** — return Brotli when client sends `br`, Gzip otherwise, raw if neither. Don't compress only the root page; JSON endpoints are hit far more often by the dashboard's 5 s auto-refresh loop.
2. **Emit `Vary: Accept-Encoding`** on every response so CDNs / browser caches / reverse proxies key the variants correctly.
3. **Add a test** asserting that the compressed variant decompresses to the same bytes as the uncompressed variant. Cheap insurance against accidentally double-compressing or skipping the wrap.
4. **Pick the right compression level** — `CompressionLevel.Optimal` for *pre-compressed* static pages (one-time CPU at startup); `CompressionLevel.Fastest` for *per-request* dynamic responses (the encoder runs on every hit).

If you're modifying `DashboardServiceCollectionExtensions.WriteJsonAsync` or `WriteCompressedHtmlAsync`, this checklist applies. If you're adding a new endpoint that uses neither, the same rules still apply — implement them inline.

### Dashboard endpoints — DI parameters that integration tests don't register

Integration tests (`tests/integration/FlowOrchestrator.Dashboard.IntegrationTests`) bootstrap the dashboard via a minimal `DashboardTestServer` that intentionally **bypasses `AddFlowOrchestrator`** — it registers only the handful of services the dashboard endpoints directly need (Hangfire in-memory, fake `IFlowStore`, etc.). Adding a new constructor / minimal-API parameter pulled from DI silently breaks every test that exercises that endpoint:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoangsnowy/FlowOrchestrator](https://github.com/hoangsnowy/FlowOrchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
