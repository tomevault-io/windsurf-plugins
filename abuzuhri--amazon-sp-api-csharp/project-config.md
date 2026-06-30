---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

`CSharpAmazonSpAPI` (NuGet id) / `FikaAmazonAPI` (assembly) — a .NET binding for Amazon's Selling Partner (SP) API. The SDK code is largely generated from Amazon's swagger/OpenAPI models (https://github.com/amzn/selling-partner-api-models) and then wrapped with hand-written `*Service` classes that handle auth, rate limiting, paging, and PII (Restricted Data Token) flows.

## Solution layout

`Amazon-SP-API-CSharp.sln` contains three buildable projects under `Source/`:

- **`FikaAmazonAPI/`** — the shipping library, `netstandard2.0`. This is the only thing that gets packed/published.
- **`FikaAmazonAPI.SampleCode/`** — `net6.0` console app with one `*Sample.cs` per API surface. Treat this as the live usage documentation; when adding/changing public methods, update the matching sample.
- **`Tests/`** — `net6.0` NUnit project. Currently only `RateLimitsTests.cs`. The library exposes `internal` members to it via `[assembly: InternalsVisibleTo("Tests")]` (see `Utils/RateLimits.cs`).

`Source/RestSharp/` is **not** a project — it's a vendored fork of RestSharp. `FikaAmazonAPI.csproj` glob-includes it via `<Compile Include="..\RestSharp\*.cs" />`. Don't add a project reference; edits there compile straight into the library.

## Build / test / pack

```bash
dotnet restore
dotnet build                                              # builds all three projects
dotnet build ./Source/FikaAmazonAPI/FikaAmazonAPI.csproj --configuration Release
dotnet test  ./Source/Tests/Tests.csproj                  # NUnit suite (rate-limit tests only)
dotnet test  ./Source/Tests/Tests.csproj --filter "FullyQualifiedName~WaitForPermittedRequest"   # single test
dotnet pack  ./Source/FikaAmazonAPI/FikaAmazonAPI.csproj  --configuration Release   # produces .nupkg
```

CI (`.github/workflows/dotnet.yml`) runs `dotnet restore` + `dotnet build --no-restore` on `ubuntu-latest` with .NET 6 — there is no test step in CI. `nuget.yml` publishes the package on tags matching `v*.*.*` or `*.*.*` (and `-rc*` variants); bump `<Version>` / `<AssemblyVersion>` / `<FileVersion>` in `FikaAmazonAPI.csproj` together when releasing.

## Architecture

### Entry point — `AmazonConnection`

`AmazonConnection` (constructed with an `AmazonCredential`) is the user-facing facade. Its constructor eagerly instantiates one `*Service` per API (`Orders`, `Reports`, `Feed`, `FbaInventory`, …) and exposes them as properties. To add a new API surface, you must touch **three** places in `AmazonConnection.cs`: the public property, the private backing field, and the `Init(...)` initialization. `ValidateCredentials` enforces ClientId/ClientSecret/RefreshToken presence and resolves `MarketPlaceID` → `MarketPlace` if only the ID was supplied.

AWS IAM (AccessKey/SecretKey/RoleArn/STS request signing) is no longer required — that branch is commented out throughout (`AmazonConnection.ValidateCredentials`, `RequestService.ExecuteRequestTry`). Don't reintroduce it without a clear reason; LWA refresh-token auth is the supported path.

### Service base — `Services/RequestService`

Every `*Service` inherits `RequestService : ApiUrls`. The lifecycle of a single SP-API call is:

1. `CreateAuthorizedRequestAsync(url, method, queryParameters, postJsonObj, tokenDataType, parameter)` — refreshes the LWA access token (or a Restricted Data Token if the parameter implements `IParameterBasedPII` and `IsNeedRestrictedDataToken == true`), then constructs the `RestRequest`.
2. `ExecuteRequestAsync<T>(rateLimitType)` wraps `ExecuteRequestTry<T>` in a retry loop bounded by `AmazonCredential.MaxThrottledRetryCount` (default 3) on `AmazonQuotaExceededException`.
3. After each response, headers are stashed in `LastHeaders`, and `SleepForRateLimit` updates the per-operation `RateLimits` bucket.

The base URL switches on `AmazonCredential.Environment` (`Sandbox` vs `Production`) via `MarketPlace.Region.SandboxHostUrl` / `HostUrl`. `RestClient` is constructed per-request with `RestClientOptions { Proxy = AmazonCredential.Proxy }` and `UseNewtonsoftJson()` — `IWebProxy` flows from credentials all the way down.

### Rate limiting — `Utils/RateLimits` + `RateLimitType`

Per-operation token-bucket limiter. `RateLimitsDefinitions.RateLimitsTime()` populates a `Dictionary<RateLimitType, RateLimits>` on each `AmazonCredential`. Services pass a `RateLimitType` enum value (e.g. `RateLimitType.Order_GetOrders`) into `ExecuteRequestAsync`. When adding a new endpoint, add a `RateLimitType` enum entry and register its rate/burst in `RateLimitsDefinitions`. Set `AmazonCredential.IsActiveLimitRate = false` to disable.

### Parameters and PII / RDT flow

Request params live under `Parameter/<Domain>/Parameter*.cs` and typically derive from `ParameterBased` (with attributes `[CamelCase]`, `[PathParameter]`, `[BodyParameter]`, `[IgnoreToAddParameter]` controlling how `getParameters()` serializes them).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abuzuhri/Amazon-SP-API-CSharp](https://github.com/abuzuhri/Amazon-SP-API-CSharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
