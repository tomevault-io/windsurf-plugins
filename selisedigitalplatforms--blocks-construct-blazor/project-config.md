---
trigger: always_on
description: This is a SELISE Blocks Blazor WASM application with the following architecture:
---

# Copilot Instructions — Blocks Construct Blazor

## Project Overview

This is a SELISE Blocks Blazor WASM application with the following architecture:

- **Client** (.NET 10 Blazor WASM): SPA frontend components and pages
- **Server** (.NET 10 Blazor Server): Backend hosting and API routes
- **Services** (.NET 10 class library): Shared business logic and service layer
- **Worker** (.NET 10 worker service): Background job processor
- **Test** (.NET 10 xUnit): Unit test projects

## Technology Stack

- **Frontend**: Blazor WASM (.NET 10) with Tailwind CSS v4 (only CSS framework — no other CSS libraries or scoped CSS)
- **Backend**: ASP.NET Core 10, GraphQL API, Swagger/OpenAPI
- **Authentication**: OIDC (SELISE Blocks identity)
- **Data**: GraphQL queries/mutations, S3 file uploads
- **CSS**: Tailwind CSS v4 (standalone CLI via MSBuild, source in `src/Server/wwwroot/app.tailwind.css`)
- **Deployment**: Docker (worker service)

---

## Render Mode — Interactive Auto (Per-Page)

This app uses **Interactive Auto with per-page rendering**. Follow these rules strictly:

### Rules

1. **Every `@page` component in `src/Client/Pages/`** MUST declare `@rendermode InteractiveAuto` at the top (line 2, after `@page`).
2. **Never set a global render mode** on `<Routes />` in `App.razor` or `Routes.razor` — the Router and MainLayout stay SSR.
3. **Non-page components** (child components, shared UI) should NOT declare `@rendermode` — they inherit from the page that uses them.
4. **Do not use `InteractiveServer` or `InteractiveWebAssembly` alone** unless there is a specific documented reason. Default is always `InteractiveAuto`.
5. **Prerendering** is on by default with `InteractiveAuto`. If a component uses `IJSRuntime` or browser-only APIs, guard calls inside `OnAfterRenderAsync(firstRender)` or disable prerendering with `@rendermode @(new InteractiveAutoRenderMode(prerender: false))`.
6. `HttpContext` is only available during static SSR — never access it in interactive components.

### Project-Level Render Mode Setup (already configured)

- `Server/Program.cs`: `AddInteractiveServerComponents()` + `AddInteractiveWebAssemblyComponents()`
- Endpoint mapping: `AddInteractiveServerRenderMode()` + `AddInteractiveWebAssemblyRenderMode()`
- Client assembly: `AddAdditionalAssemblies(typeof(Client._Imports).Assembly)`

---

## Architecture Conventions

### Folder Structure

```
src/
├── Client/
│   ├── Components/
│   │   ├── Shared/          ← reusable UI (ThemeToggle, LoadingSpinner, PageHeader, etc.)
│   │   └── Forms/           ← form-specific components
│   └── Pages/
│       └── {Feature}/       ← one folder per feature, e.g. Auth/, Dashboard/
│           └── {Feature}Page.razor
├── Server/
│   ├── Layout/              ← App.razor, MainLayout, Routes, ReconnectModal (SSR only)
│   ├── Controllers/         ← [ApiController] REST endpoints
│   └── Extensions/          ← DI registration (ServiceExtensions.cs)
├── Services/
│   └── {Feature}/           ← one folder per feature, e.g. SalesOrders/
│       ├── I{Feature}Service.cs
│       ├── {Feature}Service.cs
│       └── {Feature}.cs     ← domain model(s)
├── Test/
│   ├── Pages/               ← bUnit component tests
│   └── Services/            ← xUnit unit tests per feature
└── Worker/
    └── Jobs/                ← one class per background job
```

> **Why no `Components/` in Server?** The Server project only contains SSR shell files (`App.razor`, `MainLayout.razor`, `Routes.razor`, `ReconnectModal.razor`, `NotFound.razor`, `Error.razor`) — all placed directly in `src/Server/Layout/`. There is no `Components/` wrapper and no `Shared/` folder in Server. The `_Imports.razor` for the Server project lives at `src/Server/_Imports.razor` (project root level).

### HttpClient

- **Server project**: Use `IHttpClientFactory` (`builder.Services.AddHttpClient()`). Never register `HttpClient` with `NavigationManager.BaseUri` — it breaks during SSR.
- **Client project**: Register `HttpClient` with `builder.HostEnvironment.BaseAddress` for WASM-side API calls.
- For authenticated API calls, add a `DelegatingHandler` that injects the auth token.

#### SSR-compatible `HttpClient` (required for `InteractiveAuto` prerendering)

Client components that call APIs via `HttpClient` during `OnInitializedAsync` will run on the **server** during SSR prerender. The Client's `HttpClient` (registered with `HostEnvironment.BaseAddress`) does not resolve on the server and will silently fail.

Always register a scoped `HttpClient` in `Server/Program.cs` that uses the current request's host, so API calls work during SSR:

```csharp
// Server/Program.cs
builder.Services.AddHttpContextAccessor();
builder.Services.AddScoped<HttpClient>(sp =>
{
    var ctx = sp.GetRequiredService<IHttpContextAccessor>().HttpContext;
    var baseAddress = ctx is not null
        ? $"{ctx.Request.Scheme}://{ctx.Request.Host}/"
        : "http://localhost/";
    return new HttpClient { BaseAddress = new Uri(baseAddress) };
});
```

This replaces the plain `builder.Services.AddHttpClient()` for the scoped `HttpClient` used by Client components.

### Services Layer (`src/Services/`)

- All shared business logic goes here. Referenced by both Server and Client projects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SELISEdigitalplatforms/blocks-construct-blazor](https://github.com/SELISEdigitalplatforms/blocks-construct-blazor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
