---
trigger: always_on
description: Intune Commander is a .NET 10 / React 19 / WPF+WebView2 Windows desktop app **and CLI** for managing Microsoft Intune configurations across Commercial, GCC, GCC-High, and DoD clouds. It's a ground-up remake of a PowerShell/WPF tool — the migration to compiled .NET specifically targets UI deadlocks and threading issues.
---

# Copilot Instructions

## Project Overview
Intune Commander is a .NET 10 / React 19 / WPF+WebView2 Windows desktop app **and CLI** for managing Microsoft Intune configurations across Commercial, GCC, GCC-High, and DoD clouds. It's a ground-up remake of a PowerShell/WPF tool — the migration to compiled .NET specifically targets UI deadlocks and threading issues.

## External Documentation
- Use Context7 first for external frameworks, libraries, SDKs, GitHub Actions, and APIs whenever current behavior matters.
- Prefer Context7 for .NET, React, Zustand, Microsoft.Graph.Beta, Azure.Identity, LiteDB, PowerShell modules, and GitHub Actions before relying on memory.
- Skip Context7 only for purely repository-local code or when no relevant library entry exists.

## Critical: Async-First Rule
- **No `.GetAwaiter().GetResult()`, `.Wait()`, or `.Result` calls — ever.** Always `await`.
- All async methods must accept `CancellationToken`.

## Technology Stack
| Component | Technology |
|-----------|-----------|
| Runtime | .NET 10, C# 12 |
| Desktop Host | WPF + WebView2 (Windows-only) |
| Frontend | React 19, TypeScript, Vite |
| State Management | Zustand |
| .NET ↔ React Bridge | `ic/1` protocol via `window.chrome.webview.postMessage` |
| Authentication | Azure.Identity 1.17.x |
| Graph API | **Microsoft.Graph.Beta** 5.130.x-preview |
| Cache | LiteDB 5.0.x (encrypted via DataProtection) |
| Profile storage | `Microsoft.AspNetCore.DataProtection` |
| DI | `Microsoft.Extensions.DependencyInjection` 10.0.x |
| Testing | xUnit, NSubstitute 5.3.x |

**Important:** Uses `Microsoft.Graph.Beta` (not the stable `Microsoft.Graph`). All models and `GraphServiceClient` come from `Microsoft.Graph.Beta.*`.

## Architecture

### Projects
- `Intune.Commander.Core` — class library: auth, 30+ Graph services, models, export/import
- `Intune.Commander.DesktopReact` — WPF + WebView2 host; bridge services and `BridgeRouter`
- `Intune.Commander.CLI` — `System.CommandLine`-based CLI: `export`, `import`, `list`, `profile`, `diff`, `alert`, `completion`
- `Intune.Commander.Installer` — Master Packager Dev package (`package.json`) producing MSI + MSIX
- `intune-commander-react/` — React 19 + TypeScript frontend (Vite)

### DI and service lifetimes
`App.xaml.cs` calls `services.AddIntuneCommanderCore()` which registers:
- **Singleton:** `IAuthenticationProvider`, `IntuneGraphClientFactory`, `ProfileService`, `IProfileEncryptionService`, `ICacheService`
- **Transient:** `IExportService`

**Graph API services are NOT registered in DI.** After authentication, the WPF host (and CLI) creates them using `new XxxService(graphClient)`. Bridge services implement `IBridgeService` and are dispatched via `BridgeRouter`.

### Bridge pattern (Desktop UI)
React communicates with .NET via the `ic/1` protocol:
- **Production**: `window.chrome.webview.postMessage(msg)` → `BridgeRouter` → `IBridgeService`
- **Dev mode**: `bridgeClient.ts` falls back to a WebSocket at `ws://localhost:5100/ws/` when WebView2 is unavailable
- Messages: `{ protocol: 'ic/1', id, command, payload }` — responses keyed by `id`, push events by `event` name
- Auth commands use a 120s timeout; all others 10s

### State management
Zustand stores in `intune-commander-react/src/store/` — one store per domain (e.g., `settingsCatalogStore.ts`, `detectionRemediationStore.ts`). Each workspace has its own store.

### Caching
`CacheService` uses LiteDB at `%LocalAppData%\Intune.Commander\cache.db` (AES-encrypted). Cache key = tenant ID + data-type string. Default TTL: 24 hours.

### Profile storage
`ProfileService` stores encrypted JSON at `%LocalAppData%\Intune.Commander\profiles.json`. The file is prefixed with `INTUNEMANAGER_ENC:` when encrypted via DataProtection.

### Multi-cloud
`CloudEndpoints.GetEndpoints(cloud)` returns `(graphBaseUrl, authorityHost)`:
- Commercial & GCC → `https://graph.microsoft.com`
- GCC-High → `https://graph.microsoft.us`
- DoD → `https://dod-graph.microsoft.us`

## Service-per-Type Pattern
Each Intune object type gets its own interface + implementation. All services take `GraphServiceClient` in constructor, use manual `@odata.nextLink` pagination, accept `CancellationToken`, and return `List<T>`.

## Graph API Pagination — Manual `@odata.nextLink` (REQUIRED)
**Do NOT use `PageIterator`** — it silently truncates results on some tenants. All Graph list operations must use manual `while` loop pagination:
```csharp
var response = await _graphClient.DeviceAppManagement.MobileApps
    .GetAsync(req =>
    {
        req.QueryParameters.Top = 999;
        // other query params...
    }, cancellationToken);

var result = new List<MobileApp>();
while (response != null)
{
    if (response.Value != null)
        result.AddRange(response.Value);

    if (!string.IsNullOrEmpty(response.OdataNextLink))
    {
        response = await _graphClient.DeviceAppManagement.MobileApps
            .WithUrl(response.OdataNextLink)
            .GetAsync(cancellationToken: cancellationToken);
    }
    else
    {
        break;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamgell/IntuneCommander](https://github.com/adamgell/IntuneCommander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
