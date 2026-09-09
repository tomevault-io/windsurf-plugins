---
trigger: always_on
description: SparkplugNet is a library that implements the Sparkplug IIoT standard on top of
---

# Project rules for Claude

## What this is

SparkplugNet is a library that implements the Sparkplug IIoT standard on top of
[MQTTnet](https://github.com/dotnet/MQTTnet). It is published as the NuGet package
[SparkplugNet](https://www.nuget.org/packages/SparkplugNet/), so the library project sets
`GeneratePackageOnBuild` and the repository ships `BuildAndPushPackage.bat` for the upload. Both
Sparkplug payload versions are supported, version A (the old Kura format, namespace `spAv1.0`) and
version B (`spBv1.0`), and for version B both specification versions 2.2 and 3.0.

One solution `src/SparkplugNet.sln` with exactly three projects:

- `src/SparkplugNet/SparkplugNet.csproj`, the library, multi targeting `net8.0;net10.0`.
- `src/SparkplugNet.Tests/SparkplugNet.Tests.csproj`, MSTest, `net10.0`, 71 test methods in eight
  test classes. `dotnet test` reports 72 results, one method carries two `DataRow` attributes.
- `src/SparkplugNet.Examples/SparkplugNet.Examples.csproj`, `Exe`, `net10.0`, a console demo that
  runs an application and a node for both payload versions against `localhost:1883`.

Layout inside `src/SparkplugNet`:

- `Core/SparkplugBase.cs` and its partial files (`.Events.cs`, `.EventArgs.cs`,
  `.KnownMetricStorage.cs`): the generic base of everything, `SparkplugBase<T> where T : IMetric`.
  It owns the `IMqttClient`, the sequence and session numbers and the nested class
  `KnownMetricStorage`.
- `Core/Application/SparkplugApplicationBase.cs` plus its partial files: the SCADA host side. It
  subscribes to the whole namespace, tracks `NodeStates` and `DeviceStates` and publishes STATE.
- `Core/Node/SparkplugNodeBase.cs` plus `.Device.cs` and the partial files: the edge node side,
  including the devices hanging off a node.
- `Core/Messages/SparkplugMessageGenerator.cs`: the largest file of the repository. It builds every
  MQTT message (STATE, NBIRTH, DBIRTH, NDEATH, DDEATH, NDATA, DDATA, NCMD, DCMD), each of them once
  for version A and once for version B. New message types follow that same pairing.
- `Core/Messages/SparkplugTopicGenerator.cs`: builds and `Core/Topics/` parses the topic strings.
- `Core/PayloadHelper.cs`: protobuf-net serialization plus the byte conversion helpers. The file
  also carries the assembly level `InternalsVisibleTo("SparkplugNet.Tests")`, which is what makes
  the internal message generator testable.
- `VersionA/` and `VersionB/`: `Data/` holds the public metric types, `ProtoBuf/` the generated
  wire format types, `PayloadConverter.cs` converts between the two, and `SparkplugApplication.cs`
  and `SparkplugNode.cs` are the concrete classes a consumer instantiates.
- `GlobalUsings.cs`: all usings of the project, including the aliases `VersionAData`,
  `VersionBData`, `VersionAProtoBuf`, `VersionBProtoBuf`, `VersionADataTypeEnum`,
  `VersionBDataTypeEnum` and `SystemCancellationToken`.

Repository root: `README.md` (badges, supported frameworks, structure), `HowToUse.md` (the usage
samples for both versions), `Changelog.md`, `Updating.md` (the five step release recipe),
`Version3Compatibility.md` (the TCK rule table), `License.txt` (MIT), `Icon.png` and `Icon.svg`,
`.all-contributorsrc`, `BuildAndPushPackage.bat`, `Delete-BIN-OBJ-Folders.bat`, `.editorconfig`
(under `src`) and `.gitattributes`. `doc/` holds the specification PDFs and the two `.proto` files
the ProtoBuf classes were generated from.

## Build

```powershell
dotnet build src/SparkplugNet.sln -c Release
```

```powershell
dotnet test src/SparkplugNet.sln -c Release
```

- The library multi targets `net8.0;net10.0`, test and example project are single target
  `net10.0`. Both library targets are LTS, net8.0 goes out of support in November 2026 and is
  meant to be dropped then. The test project cannot target net8.0 on every machine, the x64
  runtime 8.0 is not necessarily installed, so the net8.0 build of the library is compiled but not
  covered by a test run.
  A release build therefore reports four projects, the library counts twice.
- `src/Directory.Build.props` contains nothing but `GenerateDocumentationFile`. Every other build
  property lives in the three `.csproj` files and is duplicated there.
- `TreatWarningsAsErrors` is enabled in all three projects, so every warning breaks the build,
  NuGet warnings (`NU****`) from restore included. A clean build reports zero warnings, keep it
  that way.
- The `NoWarn` list is `NU1803,CS0618,CS0809,NU1901,NU1902` in all three projects. `CS0618` and
  `CS0809` are there because the library keeps obsolete members alive, `NU1901` and `NU1902` hide
  low and moderate vulnerability audit findings. Fix warnings instead of extending that list.
  `NuGetAuditMode=all` is on, so a vulnerable transitive package fails the build too.
- Versions come from GitVersion.MsBuild out of the git tags, for example `1.4.2-5` for the fifth
  commit after tag `1.4.1`. There is no `GitVersion.yml`, the defaults apply. Never edit a version
  property or an assembly version by hand.
- Restore needs nuget.org. If a private feed is configured globally on the machine and answers 404
  for public packages, restore fails with `NU1301`. Then build with an explicit source:
  `dotnet build src/SparkplugNet.sln --source https://api.nuget.org/v3/index.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeppPenner/SparkplugNet](https://github.com/SeppPenner/SparkplugNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
