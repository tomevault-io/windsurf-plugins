---
trigger: always_on
description: PdfPasswordRemover is a small Windows Forms application that removes the password from a PDF file.
---

# Project rules for Claude

## What this is

PdfPasswordRemover is a small Windows Forms application that removes the password from a PDF file.
The user picks a PDF, optionally types its user password, and the app writes an unprotected copy to
a second file. The PDF handling itself is not implemented here, it comes from the NuGet package
[iTextSharp](https://www.nuget.org/packages/iTextSharp/). The repository ships a GUI application
with an installer, it is **not** published as a NuGet package: no `GeneratePackageOnBuild`, no push
script for a package.

One solution `src/PdfPasswordRemover.sln` with exactly one project:

- `src/PdfPasswordRemover/PdfPasswordRemover.csproj`, `OutputType` `WinExe`,
  `UseWindowsForms`, the whole application.

Layout inside `src/PdfPasswordRemover`:

- `Program.cs`: the `[STAThread]` entry point, `Application.Run(new Main())`.
- `Main.cs` plus `Main.Designer.cs` and `Main.resx`: the single form. `Main.cs` holds the logic,
  `Main.Designer.cs` is generated designer code, do not hand edit it against the designer. The form
  loads a PDF, and if the password box is empty it brute forces the password, otherwise it removes
  the given password. It also drives the language combo box and the show/hide password button.
- `RemovePassword.cs` plus `IRemovePassword.cs`: `CopyPdf` opens the PDF with iTextSharp and copies
  every page into a new, unprotected document. This is the one place that touches iTextSharp.
- `languages/de-DE.xml` and `languages/en-US.xml`: the translation files consumed by
  `HaemmerElectronics.SeppPenner.Language`. Both are copied next to the executable
  (`CopyToOutputDirectory=Always`).
- `GlobalUsings.cs`: all usings of the project.
- `Icon.ico` and `License.txt`: the application icon and the license shipped next to the executable.

Repository root: `README.md` (the only user documentation), `Changelog.md`, `License.txt` (MIT),
`Screenshot_DE.PNG` and `Screenshot_EN.PNG` (linked from the README), `.editorconfig` lives under
`src`, `.gitattributes` and `.gitignore` at the root. There is no `Updating.md` and no
`HowToUse.md`.

`Setup/` holds the Inno Setup script `PdfPasswordRemover-Setup.iss`, the build helper
`build-setup-files.bat` and the built installer `PdfPasswordRemover-Setup.exe`.

## Build

```powershell
dotnet build src/PdfPasswordRemover.sln
```

There are no unit tests in this repository, so there is nothing to `dotnet test`. A behaviour
change is verified by running the app and by building the installer, see the last two bullets.

- Single target framework `net10.0-windows`, no multi-targeting. The project is `WinExe` with
  `UseWindowsForms`, so it only builds on Windows. `RuntimeIdentifiers` is `win-x64`.
- All build properties live directly in `PdfPasswordRemover.csproj`. There is **no**
  `Directory.Build.props` in this repository.
- `TreatWarningsAsErrors` is enabled, so every warning breaks the build, NuGet warnings (`NU****`)
  from restore included. A clean build reports zero warnings, keep it that way.
- `NU1701`, `NU1703` and `NU1803` are suppressed via `NoWarn` (iTextSharp resolves through the
  .NET Framework compatibility shim, which raises `NU1701`). Fix warnings instead of extending that
  list. `NuGetAudit` and `NuGetAuditMode=all` are on, so a vulnerable transitive package fails the
  build too.
- Versions come from GitVersion.MsBuild out of the git tags, for example `1.0.9-1` for the first
  commit after tag `1.0.8`. Never edit a version property or an assembly version by hand. The form
  title is `Application.ProductName + " " + Application.ProductVersion`, so an untagged build shows a
  prerelease string like `PdfPasswordRemover 1.0.9-1+Branch.master.Sha...` in the window caption.
- Restore needs nuget.org. If a private feed is configured globally on the machine and answers 404
  for public packages, restore fails with `NU1301`. Then build with an explicit source:
  `dotnet build src/PdfPasswordRemover.sln --source https://api.nuget.org/v3/index.json`.
- Running the app: start `PdfPasswordRemover.exe` from the build or publish output. It is a GUI, it
  writes nothing to stdout, judge a run by the window coming up with the loaded languages, not by
  console output.
- Building the installer: `Setup/build-setup-files.bat` publishes self contained to
  `bin/publish` and removes the `*.pdb`, then Inno Setup's `ISCC.exe` compiles
  `Setup/PdfPasswordRemover-Setup.iss` into `Setup/PdfPasswordRemover-Setup.exe`.

## Code conventions

Follow the surrounding code, it is consistent throughout every file:

- File header comment block with `<copyright file="..." company="Hämmer Electronics">` and a
  `<summary>`, then the file-scoped namespace.
- XML doc comments on every type and every member, private members included, no exceptions.
  Implementations of an interface member additionally carry `<inheritdoc cref="..."/>` and
  `<seealso cref="..."/>` pointing at that interface.
- `Nullable`, `ImplicitUsings` and `LangVersion latest` are enabled.
- New `using` directives go into `GlobalUsings.cs`, inside the existing `#pragma warning disable
  IDE0065` block, never at the top of a file. The editorconfig requires usings inside the namespace

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeppPenner/PdfPasswordRemover](https://github.com/SeppPenner/PdfPasswordRemover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
