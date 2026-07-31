---
trigger: always_on
description: This document describes the project structure, conventions, and guidelines for AI agents working in this codebase.
---

# Agents Guide — Mollie API Client for .NET

This document describes the project structure, conventions, and guidelines for AI agents working in this codebase.

---

## Project Overview

This is an open-source .NET library that wraps the [Mollie REST API](https://docs.mollie.com/). It is published as two NuGet packages:

| Package | Path | Purpose |
|---|---|---|
| `Mollie.Api` | `src/Mollie.Api` | Core API client library targeting `netstandard2.0` and `net8.0` |
| `Mollie.Api.AspNet` | `src/Mollie.Api.AspNet` | ASP.NET-specific webhook helpers |

Tests live under `tests/` and samples under `samples/Mollie.WebApplication.Blazor`.

---

## Solution Structure

```
src/
  Mollie.Api/
    Client/          # One concrete client class per Mollie API resource
    Client/Abstract/ # One interface per client
    Models/          # Request and response record types, grouped by resource
    JsonConverters/  # Custom System.Text.Json converters
    Framework/       # Auth, idempotency, retry policies, JSON service
    Extensions/      # Extension methods (IEnumerable, Dictionary helpers)
    Options/         # MollieOptions, MollieClientOptions
    DependencyInjection.cs
  Mollie.Api.AspNet/
    Webhooks/        # Model binders and signature filter
tests/
  Mollie.Tests.Unit/
    Client/          # One test class per client
    Models/          # Serialisation/deserialisation tests
    Framework/
  Mollie.Tests.Integration/
samples/
  Mollie.WebApplication.Blazor/
```

---

## Key Conventions

### Language & Target

- **C# 12** (`LangVersion` is set to `12`). Use modern language features such as `required` members, primary constructors, collection expressions, and `record` types where appropriate.
- The library targets **`netstandard2.0`** (via PolySharp for back-fill) and **`net8.0`**.
- Nullable reference types are **enabled** (`<Nullable>enable</Nullable>`). Always annotate nullability correctly.

### Naming

| Artifact | Convention | Example |
|---|---|---|
| Client interface | `I{Resource}Client` | `IPaymentClient` |
| Client class | `{Resource}Client` | `PaymentClient` |
| Request model | `{Resource}Request` | `PaymentRequest` |
| Update request | `{Resource}UpdateRequest` | `PaymentUpdateRequest` |
| Response model | `{Resource}Response` | `PaymentResponse` |
| List response | `ListResponse<{Resource}Response>` | `ListResponse<PaymentResponse>` |
| Test class | `{Resource}ClientTests` | `PaymentClientTests` |

### Client Pattern

Every API resource follows this pattern:

1. **Interface** in `Client/Abstract/I{Resource}Client.cs` — inherits `IBaseMollieClient`, all public methods documented with XML `<summary>` / `<param>` / `<returns>` comments.
2. **Implementation** in `Client/{Resource}Client.cs` — inherits `BaseMollieClient`, implements the interface.
3. **Two constructors** on every concrete client:
   - `(string apiKey, HttpClient? httpClient = null)` — for manual instantiation.
   - `[ActivatorUtilitiesConstructor] (MollieClientOptions options, IMollieSecretManager mollieSecretManager, HttpClient? httpClient = null)` — used by DI.
4. **Register** the new client pair in `DependencyInjection.cs` via `RegisterMollieApiClient<IFooClient, FooClient>`.

```csharp
// Typical method signature
public async Task<FooResponse> GetFooAsync(
    string fooId,
    bool testmode = false,
    CancellationToken cancellationToken = default) {

    ValidateRequiredUrlParameter(nameof(fooId), fooId);
    var queryParameters = BuildQueryParameters(testmode: testmode);
    return await GetAsync<FooResponse>(
        $"foos/{fooId}{queryParameters.ToQueryString()}",
        cancellationToken: cancellationToken).ConfigureAwait(false);
}
```

Key rules:
- Always call `ValidateRequiredUrlParameter` for ID path segments.
- Always call `ValidateApiKeyIsOauthAccesstoken()` when a parameter requires an OAuth token (e.g., `profileId`, `testmode`, `applicationFee`).
- Always pass `cancellationToken` through and call `.ConfigureAwait(false)` on every `await`.
- Use the protected helpers `GetAsync`, `PostAsync`, `PatchAsync`, `DeleteAsync`, `GetListAsync` from `BaseMollieClient` — never call `HttpClient` directly.
- Build query strings using the `BuildQueryParameters` helper and the `ToQueryString()` extension method.

### Model Pattern

- **Request** and **Response** types are `record` types.
- Required fields are annotated with the `required` keyword.
- Optional fields are nullable (`string?`, `DateTime?`, etc.).
- All properties have XML `<summary>` documentation comments.
- Use `[JsonPropertyName("...")]` only when the JSON key differs from the C# property name.
- Use `[JsonConverter(typeof(RawJsonConverter))]` for `Metadata` fields that hold raw JSON.
- JSON is handled by `System.Text.Json` — do **not** add a dependency on `Newtonsoft.Json`.
- Response models that implement `IEntity` expose an `Id` property.
- Embedded resources are placed in a nested `{Resource}EmbeddedResponse` type annotated with `[JsonPropertyName("_embedded")]`.
- HAL-style link objects are placed in a nested `{Resource}ResponseLinks` type annotated with `[JsonPropertyName("_links")]` and typed as `UrlObjectLink<T>`.

### Extensions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Viincenttt/MollieApi](https://github.com/Viincenttt/MollieApi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
