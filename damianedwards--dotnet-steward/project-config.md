---
trigger: always_on
description: This file applies to the entire repository.
---

# DotNetSteward contributor guidance

This file applies to the entire repository.

## Project purpose

DotNetSteward is a Windows-only PowerShell module for discovering, installing,
inventorying, and updating official Microsoft .NET SDK and runtime
installations managed by Windows installers.

Keep these product boundaries intact:

- Support Windows PowerShell 5.1 and PowerShell 7+.
- Manage official Windows EXE/MSI installer registrations, not ZIP/archive,
  repository-local, or daily-build installations.
- Support `Sdk`, `Runtime`, `AspNetCoreRuntime`, and
  `WindowsDesktopRuntime` across x64, x86, and Arm64.
- Treat standalone EXE bundles as actionable.
- Keep MSI payloads owned by an SDK, Visual Studio, or another installer
  visible but read-only.

## Public commands

The exported commands are:

- `Find-DotNetSdk`
- `Find-DotNetRuntime`
- `Get-DotNetInstallation`
- `Install-DotNetSdk`
- `Install-DotNetRuntime`
- `Update-DotNetSdk`
- `Update-DotNetRuntime`

`Find-*` queries the official remote release catalog. `Get-*` reports local
Windows Installer state. `Install-*` performs fresh installations without
requiring an existing .NET installation. `Update-*` starts from actionable
local bundle registrations.

When adding or renaming a public command:

1. Put it in a same-named file under `Public`.
2. Include comment-based help and representative examples.
3. Add it to the deterministic loader and `Export-ModuleMember` in
   `DotNetSteward.psm1`.
4. Add it to `FunctionsToExport` in `DotNetSteward.psd1`.
5. Update the expected export list in `tests\Verify.ps1`.
6. Document its user-facing behavior in `README.md`.

## Architecture

- `DotNetSteward.psm1`: strict-mode module loader, trust constants, and exports.
- `DotNetSteward.psd1`: Gallery metadata and public API declaration.
- `DotNetSteward.Format.ps1xml`: default views for installations and available
  releases.
- `Public`: exported advanced functions only.
- `Private\Discovery.ps1`: registry inventory and ownership resolution.
- `Private\ReleaseMetadata.ps1`: official catalog and installer metadata.
- `Private\AvailableReleases.ps1`: remote selection and fresh-install planning.
- `Private\UpdatePlanning.ps1`: installed-product update selection.
- `Private\Versioning.ps1`: .NET/SemVer-compatible version parsing and ordering.
- `Private\Downloads.ps1`: parallel BITS downloads.
- `Private\InstallerTrust.ps1`: hash and Authenticode enforcement.
- `Private\Installation.ps1`: sequential elevated installer execution.
- `tests\Verify.ps1`: cross-edition parser, API, inventory, and behavior checks.
- `scripts`: release versioning, staging, packaging, and artifact verification.

Prefer extending the relevant responsibility-focused file over adding logic to
the root module or duplicating helpers in public commands.

## Inventory invariants

Windows Installer registrations are the source of truth for local inventory;
do not switch discovery back to `dotnet --info`.

- Standalone Burn bundles are read from the 32-bit HKLM uninstall registry
  view. This view contains x64, x86, and Arm64 bundle registrations and is not
  limited to x86 payloads.
- Hidden MSI payload records are read from both 32-bit and 64-bit uninstall
  views.
- Ownership is resolved through
  `HKLM\SOFTWARE\Classes\Installer\Dependencies`.
- `VS.{AEF703B8-D2CC-4343-915C-F54A30B90937}` identifies the Visual Studio
  dependent.
- Anything managed by Visual Studio or shared with Visual Studio must remain
  non-updateable and non-uninstallable.
- Never query `Win32_Product`; it is slow and can trigger MSI consistency
  checks or repairs.

Preserve logical deduplication when the same payload appears in multiple
registry views or has multiple owners.

## Release selection

Use only Microsoft's official releases index:

`https://builds.dotnet.microsoft.com/dotnet/release-metadata/releases-index.json`

Daily builds are intentionally unsupported. Release catalog operations must:

- accept only absolute HTTPS metadata and installer URLs;
- select installers by exact RID and expected EXE name;
- expose only entries with a usable SHA-256 or SHA-512 hash;
- preserve the catalog channel separately from the SDK version, because early
  SDK and runtime channel numbers did not always align;
- use the custom version helpers rather than `[version]` for prerelease
  ordering;
- keep exact prerelease requests explicit, while requiring `-IncludePreview`
  for broad stable-to-preview selection;
- return deterministic, descending version order from `Find-*`;
- install selected versions sequentially from lowest to highest.

The version comparator must retain SemVer precedence for dot-separated numeric
and alphanumeric prerelease identifiers. Do not replace it with lexical
sorting.

## Installer security

Do not weaken installer verification or turn failures into warnings.

Before execution, every installer must:

1. Match the SHA-256 or SHA-512 hash supplied by official release metadata.
2. Have a valid Authenticode signature.
3. Match one of the expected current or legacy Microsoft leaf subjects and
   simple names configured in `DotNetSteward.psm1`.
4. Chain through an explicitly allowed Microsoft code-signing intermediate.
5. Chain to the pinned Microsoft root subject and thumbprint.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DamianEdwards/dotnet-steward](https://github.com/DamianEdwards/dotnet-steward) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
