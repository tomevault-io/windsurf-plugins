---
trigger: always_on
description: > Agent navigator for this repo. Read this first. Defer to [Docs/](Docs/) for deep how-tos (creating editors, services, migrations, admin pages).
---

# CLAUDE.md — ZauberCMS

> Agent navigator for this repo. Read this first. Defer to [Docs/](Docs/) for deep how-tos (creating editors, services, migrations, admin pages).

## Project snapshot

**ZauberCMS** is a fully-featured .NET 10 Blazor CMS, loosely modelled on Umbraco. (see [ZauberCMS.Core/ZauberCMS.Core.csproj:9](ZauberCMS.Core/ZauberCMS.Core.csproj#L9)).

- **Front-end** = static SSR with optional `@rendermode` opt-ins.
- **Admin** (`/admin`) = `InteractiveServer` Blazor + Radzen 8.4.0.
- Distributed as **4 NuGet packages** (`ZauberCMS.Core`, `ZauberCMS.Components`, `ZauberCMS.Routing`, `ZauberCMS`) + a **`dotnet new` template** (`ZauberCMS.Template`).
- **CI/CD**: every push to `main-static` runs [.github/workflows/ci.yml](.github/workflows/ci.yml) — auto-bumps the patch version, builds + packs all 5 NuGet packages, pushes to nuget.org, and creates a `vX.Y.Z` tag. [release.ps1](release.ps1) is kept for offline/local releases but is no longer the primary path.
- See [ReadMe.md](ReadMe.md) for the public pitch and screenshots; see [www.zaubercms.com](https://www.zaubercms.com/) and the GitBook docs ([aptitude.gitbook.io/zaubercms](https://aptitude.gitbook.io/zaubercms)) for end-user material.

---

## 🚨 Critical rules for agents

### 1. Never recommend building from source

The [ZauberCMS.Web/](ZauberCMS.Web/) project is a **local dev runner only**. Per [ReadMe.md:41](ReadMe.md#L41): *"DO NOT USE THE SOURCE CODE TO BUILD YOUR SITE. USE THE TEMPLATE OR NUGET PACKAGE!"*. Always point users at:

```ps
dotnet new install ZauberCMS.Template@4.1.0 --force
dotnet new zaubercms -n YourSiteName
```

### 2. Schema changes need three migrations — not one

Three EF Core providers, three DbContexts, three migration folders. Use [migrations.ps1](migrations.ps1) — it generates all three migrations in one shot. Never hand-roll a single provider; the others will drift.

### 3. Render mode is load-bearing

| Surface | Render mode | UI library |
| --- | --- | --- |
| Front-end content views (`IContentView`) | **`null` = static SSR** | Consumer's choice (Bootstrap, Tailwind, etc.) |
| Admin (`/admin`) | **`InteractiveServer`** | Radzen.Blazor + Blazored.Modal |

Adding interactive code to a front-end view silently breaks SSR. To opt in, mark the specific child component `@rendermode="InteractiveServer"`. **Never use Radzen on the front-end.** The split is enforced in `App.razor` based on whether the path starts with `/admin`.

### 4. Extension discovery is by interface — wrong interface = silently invisible

Property editors, views, plugins, providers, sections, and seed data are **not registered in DI by hand**. They are reflected over by `ExtensionManager` from the assemblies registered in `AssemblyManager` at startup. A class that doesn't implement the right interface (or lives in an assembly that wasn't registered) **just doesn't appear** — there is no error. Discovery interfaces:

| Interface | What it is |
| --- | --- |
| `IContentProperty` | Property editor (admin) |
| `IContentView` | Front-end view template (alias = class name) |
| `IStartupPlugin` | Adds services to DI at boot |
| `IStorageProvider` | Media storage backend (default `DiskStorageProvider`) |
| `IEmailProvider` | Email backend (default `SmtpEmailProvider`) |
| `IExternalAuthenticationProvider` | OAuth providers (Google/Facebook/Microsoft already shipped) |
| `ISection`, `ISectionNav` | Admin nav sections and items |
| `ISeedData` | Startup data seeding |

If you add one of these and it doesn't show up, check the interface and the assembly first — not DI registration.

---

## Solution layout

| Project | Path | Role | Packed? |
| --- | --- | --- | --- |
| ZauberCMS.Core | [ZauberCMS.Core/](ZauberCMS.Core/) | Domain, services, EF Core, plugins, providers, settings, seed data | ✅ NuGet |
| ZauberCMS.Components | [ZauberCMS.Components/](ZauberCMS.Components/) | Razor class library — admin UI, property editors, dialogs, layouts | ✅ NuGet |
| ZauberCMS.Routing | [ZauberCMS.Routing/](ZauberCMS.Routing/) | Front-end catch-all routing (`@page "/{**slug}"`) | ✅ NuGet |
| ZauberCMS | [ZauberCMS/](ZauberCMS/) | Umbrella: `App.razor`, `Routes.razor`, packaged `appsettings.json`, MSBuild targets | ✅ NuGet |
| ZauberCMS.Template | [ZauberCMS.Template/](ZauberCMS.Template/) | The `dotnet new zaubercms` template package | ✅ NuGet (template) |
| ZauberCMS.Web | [ZauberCMS.Web/](ZauberCMS.Web/) | **Local dev runner only.** SQLite (`app.db`). `<ProjectReference>` to ZauberCMS. | ❌ |
| ZauberCMS.StarterSite | [ZauberCMS.StarterSite/](ZauberCMS.StarterSite/) | **Stale.** Only `bin/`/`obj/` left, not in `.sln`. **Ignore.** | ❌ |

Other top-level items: [Docs/](Docs/) (authoritative dev docs — see table at bottom), [NugetSource/](NugetSource/) (local nupkg drop / feed), [release.ps1](release.ps1), [migrations.ps1](migrations.ps1), [misc.txt](misc.txt) (notes — partly outdated).

---

## Architecture

### Bootstrap entry points

Both [ZauberCMS.Web/Program.cs](ZauberCMS.Web/Program.cs) and the template's [ZauberCMS.Template/template/ZauberCMSTemplate.Site/Program.cs](ZauberCMS.Template/template/ZauberCMSTemplate.Site/Program.cs) call:

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YodasMyDad/ZauberCMS](https://github.com/YodasMyDad/ZauberCMS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
