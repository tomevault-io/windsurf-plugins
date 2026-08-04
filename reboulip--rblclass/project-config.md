---
trigger: always_on
description: This is a refactor of a legacy Outlook VBA macro into a modern Outlook
---

# CLAUDE.md

## Project overview

This is a refactor of a legacy Outlook VBA macro into a modern Outlook
COM add-in. It helps users quickly classify emails into folders across
multiple .pst archives, with fast keyword search over the folder tree,
attachment management, and send-time guards.

## Stack and constraints

- **Add-in technology**: Outlook **Shared COM Add-in** —
  `Extensibility.IDTExtensibility2` + `Office.IRibbonExtensibility`,
  registered into Outlook via HKCU keys under
  `Software\Microsoft\Office\Outlook\Addins\`. This is the classic,
  pre-VSTO Outlook add-in model. It supports PST access, offline use,
  and per-user install without admin rights on the classic Win32
  Outlook client. VSTO is explicitly excluded for this project after
  the Office Developer Tools workload could not be set up on the dev
  machine.
- **Target Outlook** (deployment): classic Win32 Outlook, **32-bit**,
  Microsoft 365 Apps for Enterprise on the **Semi-Annual Enterprise
  Channel**. NOT the "new Outlook". NO Exchange Online dependency.
- **Dev Outlook**: classic Win32 Outlook, **64-bit**, **Current**
  channel, on **Windows 11 ARM64** (Outlook runs as emulated x64
  via Prism). The local dev machine differs from the deployment
  target in both architecture and channel. The same AnyCPU
  add-in binary must load on both. Final EDR and compatibility
  validation happens against a real 32-bit target workstation;
  the dev machine is for fast iteration only.
- **Runtime**: .NET Framework 4.8 for the add-in shell and the
  Outlook adapter. .NET Standard 2.0 for the business core.
- **Language**: C# 7.3 (.NET Framework 4.8 limit unless LangVersion is
  explicitly bumped, which we avoid).
- **UI**: WPF with a **hand-rolled minimal MVVM** (`RBLclass.AddIn/Mvvm`:
  `ObservableObject` + `RelayCommand`), hosted in Custom Task Panes via
  `ICustomTaskPaneConsumer`/`ICTPFactory` — the WPF view is bridged into a
  ComVisible WinForms host control through `ElementHost`. Ribbon via
  `IRibbonExtensibility.GetCustomUI` returning Ribbon XML (not the
  Ribbon Designer). **CommunityToolkit.Mvvm is deliberately NOT used:** its
  8.x dependencies (`System.Memory` ≥ 4.5.5, `System.Runtime.CompilerServices.Unsafe`
  ≥ 6.0.0) float the SQLite facade assemblies above what `SQLitePCLRaw` was
  built against, and a COM host has no binding redirects, so it breaks the
  first `SqliteConnection` at runtime. Keep new AddIn dependencies
  dependency-light for the same reason.
- **Storage**: SQLite via Microsoft.Data.Sqlite, with FTS5 for full-text
  search. Database in %LocalAppData%\RBLclass\.
- **Logging**: Serilog with a rolling file sink.
- **Tests**: xUnit, FluentAssertions, NSubstitute. Tests run against
  RBLclass.Core only; the Outlook adapter is excluded from CI tests.
- **Packaging**: per-user install via a signed installer that lays
  down the add-in DLL + dependencies under `%LocalAppData%\RBLclass\`
  and writes the COM registration entries to HKCU. Phase 0 POC uses a
  PowerShell installer; Phase 1 promotes to an MSI built with the WiX
  Toolset, Authenticode-signed (both the DLL and the MSI) with an
  internal-PKI certificate carrying the Code Signing EKU
  (1.3.6.1.5.5.7.3.3). ClickOnce is not used — it is VSTO-specific
  in this scenario.
- **Min Windows**: Windows 10 1809.

## Architecture

Strict layering. Dependencies point downward only.

RBLclass.AddIn               (.NET FW 4.8) — COM add-in shell:
       │                                  IDTExtensibility2, IRibbonExtensibility,
       │                                  ribbon callbacks, Custom Task Panes,
       │                                  Outlook event subscriptions
RBLclass.Outlook.Adapter     (.NET FW 4.8) — COM access to Outlook OM
       │                                  Implements RBLclass.Core interfaces
       ▼
RBLclass.Core                (.NET Standard 2.0) — business logic, no Outlook
                                                no UI, no I/O except via
                                                injected interfaces


The business core (`RBLclass.Core`) MUST remain free of any reference to
Microsoft.Office.Interop.Outlook, System.Windows, or COM-add-in shell
assemblies. This is what makes the code portable on the day we have to
migrate away from the COM add-in model.

## Critical coding rules

### COM interop interface declarations (add-in shell only)

- **Never hand-roll `[ComImport]` declarations of `IDTExtensibility2`,
  `IRibbonExtensibility`, `IRibbonControl`, or other Office/Extensibility
  interfaces.** Reference the canonical PIAs from the GAC instead:
  - `Extensibility` —
    `C:\Windows\assembly\GAC\Extensibility\7.0.3300.0__b03f5f7f11d50a3a\extensibility.dll`
  - `office` (Microsoft.Office.Core) —
    `C:\Windows\assembly\GAC_MSIL\office\15.0.0.0__71e9bce111e9429c\OFFICE.DLL`
  - `Microsoft.Office.Interop.Outlook` —
    `C:\Windows\assembly\GAC_MSIL\Microsoft.Office.Interop.Outlook\15.0.0.0__71e9bce111e9429c\Microsoft.Office.Interop.Outlook.dll`

  Reference them in the `.csproj` via `<Reference Include="…"><HintPath>…</HintPath><Private>false</Private></Reference>`.
  **Why:** CLR auto-generates IL marshalling stubs from interface
  metadata. Even with the correct `[Guid]` and `[DispId]` attributes,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reboulip/rblclass](https://github.com/reboulip/rblclass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
