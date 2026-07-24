---
trigger: always_on
description: BlazorWebFormsComponents is a library that provides Blazor components emulating ASP.NET Web Forms controls, enabling migration from Web Forms to Blazor with minimal markup changes. The project helps developers reuse their existing Web Forms markup in Blazor applications.
---

# Copilot Instructions for BlazorWebFormsComponents

## Project Overview

BlazorWebFormsComponents is a library that provides Blazor components emulating ASP.NET Web Forms controls, enabling migration from Web Forms to Blazor with minimal markup changes. The project helps developers reuse their existing Web Forms markup in Blazor applications.

## Core Requirements

**ASP.NET Web Forms components that ship with .NET Framework 4.8 should be recreated as Blazor Components with:**

1. **Same Name** - The Blazor component must have the identical name as the original Web Forms control (e.g., `<asp:Button>` becomes `<Button>`, `<asp:GridView>` becomes `<GridView>`)

2. **Same Attributes and Properties** - Support the same attribute names and property signatures as the original control wherever possible. This enables developers to migrate markup with minimal changes (removing only `asp:` prefix and `runat="server"`)

3. **Identical HTML Output** - The rendered HTML must match what the original Web Forms control produces. This ensures:
   - Existing CSS styles continue to work
   - JavaScript that targets the HTML structure remains functional
   - Visual appearance is preserved after migration

### What This Means in Practice

```html
<!-- Original Web Forms -->
<asp:Button ID="btnSubmit" Text="Submit" CssClass="btn-primary" OnClick="Submit_Click" runat="server" />

<!-- Blazor Equivalent (should render identical HTML) -->
<Button Text="Submit" CssClass="btn-primary" OnClick="Submit_Click" />
```

Both should render:
```html
<button type="submit" class="btn-primary">Submit</button>
```

### Reference Documentation
- Original Web Forms controls: https://docs.microsoft.com/en-us/dotnet/api/system.web.ui.webcontrols?view=netframework-4.8
- Use the .NET Framework 4.8 reference source to verify HTML output patterns

## Technology Stack

- **.NET Version**: .NET 10.0 (see `global.json` for SDK version)
- **Framework**: Blazor (migration target is .NET 10 static SSR; samples also include Server-Side and WebAssembly)
- **Project Type**: Razor Class Library (`Microsoft.NET.Sdk.Razor`)
- **Testing Framework**: xUnit with bUnit for Blazor component testing
- **Assertion Library**: Shouldly
- **Mocking**: Moq
- **Build/Versioning**: Nerdbank.GitVersioning
- **CSS Utilities**: BlazorComponentUtilities

## Project Structure

```
/docs                                 -- User documentation (MkDocs)
/samples                              -- Usage samples and migration benchmarks (ordered by complexity ↓)
  BeforeWebForms/                     -- Original Web Forms sample (.NET Framework)
  AfterBlazorServerSide/              -- Blazor Server-Side component samples
  AfterBlazorServerSide.Tests/        -- Playwright tests for the sample app
  AfterBlazorClientSide/              -- Blazor WebAssembly component samples
  SharedSampleObjects/                -- Shared models/data for samples
  WingtipToys/                        -- ★ Current benchmark focus — e-commerce app (products, cart, checkout, identity)
  AfterWingtipToys/                   -- Migrated WingtipToys output (cleared + regenerated each benchmark run)
  ContosoUniversity/                  -- Next benchmark — university CRUD app (EF, master-detail, search)
  AfterContosoUniversity/             -- Migrated Contoso University output
  DepartmentPortal/                   -- Most complex benchmark — multi-page portal (advanced data binding, nested controls)
  AfterDepartmentPortal/              -- Migrated DepartmentPortal output
/src
  BlazorWebFormsComponents/           -- Main component library
  BlazorWebFormsComponents.Test/      -- Unit tests with bUnit
  BlazorWebFormsComponents.Analyzers/ -- Roslyn analyzers
  BlazorWebFormsComponents.Analyzers.Test/ -- Analyzer tests
  BlazorWebFormsComponents.Cli/       -- Web Forms to Blazor CLI pipeline and scaffolding
  BlazorAjaxToolkitComponents/        -- AJAX Toolkit component library
  WingtipToys.AcceptanceTests/        -- WingtipToys Playwright acceptance suite
  ContosoUniversity.AcceptanceTests/  -- ContosoUniversity Playwright acceptance suite
/tests
  BlazorWebFormsComponents.Cli.Tests/ -- CLI transform and pipeline tests
/migration-toolkit                    -- Wrapper scripts, skills, methodology, and benchmark helpers
  scripts/                            -- bwfc-migrate.ps1, bwfc-scan.ps1, and compatibility wrappers
  skills/                             -- bwfc-migration, bwfc-identity-migration, bwfc-data-migration, migration-standards, l3-performance-optimization
```

## Component Architecture

### Base Classes Hierarchy

1. **`BaseWebFormsComponent`** - Root base class for all components
   - Inherits from `ComponentBase`
   - Implements `IAsyncDisposable`
   - Provides cascading parent component support
   - Defines obsolete Web Forms properties (`ID`, `ViewState`, `runat`)
   - Provides `Enabled`, `TabIndex`, `Visible` parameters

2. **`BaseStyledComponent`** - Adds styling support
   - Inherits from `BaseWebFormsComponent`
   - Implements `IStyle`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FritzAndFriends/BlazorWebFormsComponents](https://github.com/FritzAndFriends/BlazorWebFormsComponents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
