---
trigger: always_on
description: Circuids.Bridge is a cross-platform Blazor library that abstracts host environment, platform, form factor, connectivity, theme, and safe area detection into a unified service layer. The same Razor components and services work identically across Blazor Server, Blazor WebAssembly, and MAUI Blazor Hybrid.
---

# GitHub Copilot Instructions — Circuids.Bridge

## Project Overview

Circuids.Bridge is a cross-platform Blazor library that abstracts host environment, platform, form factor, connectivity, theme, and safe area detection into a unified service layer. The same Razor components and services work identically across Blazor Server, Blazor WebAssembly, and MAUI Blazor Hybrid.

---

## Repository Structure

```
src/
  Circuids.Bridge/            # Core library: interfaces, enums, records, components, handlers
  Circuids.Bridge.Blazor/     # Blazor implementation: JS interop adapters
  Circuids.Bridge.Maui/       # MAUI implementation: native platform adapters
sample/
  Circuids.Bridge.Shared.Sample/          # Shared RCL with all sample pages
  Circuids.Bridge.Blazor.Server.Sample/   # Thin Blazor Server shell
  Circuids.Bridge.Blazor.WebAssembly.Sample/ # Thin WASM shell
  Circuids.Bridge.Maui.Sample/            # Thin MAUI Blazor Hybrid shell
docs/                         # Architecture docs, testing proposal, API reference
```

---

## Critical Coding Rules

### No C# Primary Constructors
**Never use C# primary constructors.** Always declare explicit traditional constructors with a body.

```csharp
// CORRECT
public sealed class MyService : IMyService
{
    private readonly IJSRuntime _jsRuntime;

    public MyService(IJSRuntime jsRuntime)
    {
        _jsRuntime = jsRuntime;
    }
}

// WRONG — do not use primary constructors
public sealed class MyService(IJSRuntime jsRuntime) : IMyService
{
}
```

### No C# Records for Mutable State
Use `sealed record` only for immutable value objects (e.g., `FormFactorInfo`, `SafeAreaInsets`). Use `sealed class` for services and configuration objects.

---

## Namespace Conventions

| Project | Namespace |
|---|---|
| `Circuids.Bridge` | `Circuids.Bridge` |
| `Circuids.Bridge.Blazor` — public | `Circuids.Bridge.Blazor` |
| `Circuids.Bridge.Blazor` — internal | `Circuids.Bridge.Blazor.Internal` |
| `Circuids.Bridge.Maui` — public | `Circuids.Bridge.Maui` |
| `Circuids.Bridge.Maui` — internal | `Circuids.Bridge.Maui.Internal` |

Always put `@namespace Circuids.Bridge` at the top of `.razor` files in the core library.

---

## Core Interfaces and Their Contracts

All five services live in the `Circuids.Bridge` namespace:

- **`IBridge`** — host + platform detection. Properties: `Host Host`, `PlatformIdentity Platform`, `string PlatformVersion`, `bool IsInitialized`. Event: `PlatformChanged`. Method: `InitializeAsync()`.
- **`IBridgeFormFactor`** — form factor + viewport. Property: `FormFactorInfo FormFactor`. Event: `FormFactorChanged`. Methods: `InitializeAsync(ResizeMode)`, `CreateListenerAsync()`, `DisposeListenerAsync()`.
- **`IBridgeConnectivity`** — online status. Property: `bool IsConnected`. Event: `ConnectionChanged`. Method: `InitializeAsync(ConnectivityOptions?)`.
- **`IBridgeTheme`** — light/dark mode. Property: `ThemeMode Theme`. Event: `ThemeChanged`. Method: `InitializeAsync()`.
- **`IBridgeSafeArea`** — notch/cutout insets. Property: `SafeAreaInsets SafeArea`. Event: `SafeAreaChanged`. Method: `InitializeAsync()`.

---

## Common Types

### Enums
- `Host`: `Unknown | Maui | Blazor | Wpf | WinForms`
- `PlatformIdentity`: `Unknown | Android | IOS | Windows | Mac | Linux`
- `FormFactor`: `Unknown | Phone | Tablet | Desktop`
- `ThemeMode`: `Unknown | Light | Dark`
- `ResizeMode`: `None | Global | Once`

### Records (immutable value objects)
- `FormFactorInfo(FormFactor, double Width, double Height)` — factory: `FormFactorInfo.Unknown()`
- `SafeAreaInsets(double Top, double Right, double Bottom, double Left)` — static: `SafeAreaInsets.Zero`

### Classes
- `ConnectivityOptions` — `int IntervalInSeconds = 10`, `string TestUrl = "/favicon.ico"`
- `BridgeException` — the only exception type thrown by Bridge. Extends `Exception`.

---

## DI Registration Pattern

All five services are always registered as **Scoped**.

```csharp
// Blazor
services.AddScoped<IBridge, BridgeBlazor>();
services.AddScoped<IBridgeFormFactor, BridgeFormFactorBlazor>();
services.AddScoped<IBridgeConnectivity, BridgeConnectivityBlazor>();
services.AddScoped<IBridgeTheme, BridgeThemeBlazor>();
services.AddScoped<IBridgeSafeArea, BridgeSafeAreaBlazor>();

// MAUI — same pattern but Maui* implementations
```

Public extension methods live in the `Extensions/` subfolder:
- `AddBridgeForBlazor(this IServiceCollection)` in `Circuids.Bridge.Blazor`
- `AddBridgeForMaui(this IServiceCollection)` in `Circuids.Bridge.Maui`

---

## Blazor Adapter Pattern (JS Interop)

Every Blazor internal adapter follows this exact pattern:

```csharp
internal sealed class BridgeFooBlazor : IBridgeFoo, IAsyncDisposable
{
    private readonly Lazy<Task<IJSObjectReference>> _moduleTask;
    private const string ModulePath = "./_content/Circuids.Bridge.Blazor/BridgeFoo.js";

    private bool _isInitialized;

    public BridgeFooBlazor(IJSRuntime jsRuntime)
    {
        _moduleTask = new(() => jsRuntime.InvokeAsync<IJSObjectReference>("import", ModulePath).AsTask());
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Circuids/Bridge](https://github.com/Circuids/Bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
