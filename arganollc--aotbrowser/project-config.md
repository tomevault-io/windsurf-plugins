---
trigger: always_on
description: - Treat `Projects\build-aotbrowser.yml` as the source of truth for CI. The pipeline builds `Projects\BuildSolution\BuildSolution.sln`; `Projects\AOTBrowser\AOTBrowser.sln` is the local developer solution with the same X++ model plus the support library.
---

# Copilot instructions

## Build, test, and validation

- Treat `Projects\build-aotbrowser.yml` as the source of truth for CI. The pipeline builds `Projects\BuildSolution\BuildSolution.sln`; `Projects\AOTBrowser\AOTBrowser.sln` is the local developer solution with the same X++ model plus the support library.
- Restore the D365FO build tool packages and C# NuGet packages before building:

```powershell
nuget install "Projects\packages.config" -ExcludeVersion -OutputDirectory "<NuGetsPath>" -ConfigFile "Projects\nuget.config"
nuget restore "Projects\BuildSolution\BuildSolution.sln"
```

- Full build, following the same shape as CI:

```powershell
msbuild "Projects\BuildSolution\BuildSolution.sln" /p:BuildTasksDirectory="<NuGetsPath>\Microsoft.Dynamics.AX.Platform.CompilerPackage\DevAlm" /p:MetadataDirectory="<repo>\Metadata" /p:FrameworkDirectory="<NuGetsPath>\Microsoft.Dynamics.AX.Platform.CompilerPackage" /p:ReferenceFolder="<NuGetsPath>\Microsoft.Dynamics.AX.Platform.DevALM.BuildXpp\ref\net40;<NuGetsPath>\Microsoft.Dynamics.AX.Application.DevALM.BuildXpp\ref\net40;<NuGetsPath>\Microsoft.Dynamics.AX.ApplicationSuite.DevALM.BuildXpp\ref\net40;<repo>\Metadata;<BinariesPath>" /p:ReferencePath="<NuGetsPath>\Microsoft.Dynamics.AX.Platform.CompilerPackage" /p:OutputDirectory="<BinariesPath>"
```

- If you only changed the C# helper library, build it directly:

```powershell
msbuild "Projects\AOTBrowser\Arbela.Dynamics.AX.Xpp.Support\Arbela.Dynamics.AX.Xpp.Support.csproj" /p:Configuration=Debug
```

- For local onebox development, mount the repo metadata into `PackagesLocalDirectory` with:

```powershell
powershell -ExecutionPolicy Bypass -File "Scripts\Mount.ps1"
```

  Run it as admin with Visual Studio closed. The script creates symbolic links for each model under `Metadata\` and restarts the D365FO environment.

- No automated unit-test, single-test, or lint command is checked in. Validation is feature-level in a D365FO environment: repopulate AOT data, then exercise the AOT browser form, the form-personalization entry point, the data-entity entry point, table browser, source-code toggle, inline extensions toggle, and jump-to-reference flows.

## High-level architecture

- This repository is a D365FO ISV model plus a C# interop library. The checked-in source of truth is under `Metadata\AOTBrowser\AOTBrowser\Ax*`; the `.rnrproj` files in `Projects\` are Visual Studio/build wrappers around that metadata.
- The main product surface is `Metadata\AOTBrowser\AOTBrowser\AxForm\ARBAOTBrowser.xml`. It builds a tree/detail view for a selected AOT object and switches behavior by object type through `ARBAOTObjectTree`.
- `ARBAOTObjectTree` is the core metadata-browser engine. It reads metadata through Dynamics metadata providers, builds the tree UI, merges extensions into the tree when requested, and preserves selection with tree paths.
- Object-type-specific tree behavior is split into subclasses such as `ARBAOTObjectTreeForm` and `ARBAOTObjectTreeTable`. Those subclasses handle details like form-control selection and relation rendering that do not belong in the base class.
- `ARBAOTObjectPopulateService` populates `ARBAOTObjects`, which is the persistent cache of browsable AOT root objects. `ARBEntityFieldPopulateService` populates `ARBEntityFields`, and `ARBMenuItemRefPopulateService` populates `ARBMenuItemReferences`. Those two tables drive the "open from entity field" and "jump from menu item" workflows.
- `ARBAOTObjectProperties` and `ARBAOTObjectPropertiesMatrix` are `InMemory` tables used for the current selection and comparison-style details grid; they are not persistent metadata storage.
- The C# project in `Projects\AOTBrowser\Arbela.Dynamics.AX.Xpp.Support\` fills gaps in the X++ metadata APIs. `MetadataSupport.cs` exposes extension and source-text helpers, while `HtmlFormatHelper.cs` and `XPlusPlusLexer.cs` provide syntax-highlighted X++ rendering for the source-code pane.
- Entry points are intentionally spread across menu extensions and form extensions: `CommonMenu.AOTBrowser`, `navpanemenu.AOTBrowser`, `DMFEntity.AOTBrowser`, `FormControlPersonalization.AOTBrowser`, and `FormControlPersonalizationForm_ARB_Extension`. Changes to launch/navigation behavior usually touch more than one artifact.
- The model descriptor is `Metadata\AOTBrowser\Descriptor\AOTBrowser.xml`. It allows customization and references `ApplicationFoundation`, `ApplicationPlatform`, `ApplicationSuite`, and `GeneralLedger`.

## Key conventions

- X++ artifacts use the `ARB` / `ARBAOT` prefix. The helper library namespaces are `Arbela.Dynamics.AX.Xpp.Support` and `Arbela.Dynamics.Ax.Xpp`.
- Add object-type-specific browser behavior by creating an `ARBAOTObjectTree` subclass with an `[ARBAOTObjectType(...)]` attribute. The base class resolves implementations through `SysExtensionAppClassFactory`; prefer that extension point over adding more type-specific branching in the form.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arganollc/aotbrowser](https://github.com/arganollc/aotbrowser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
