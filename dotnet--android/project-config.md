---
trigger: always_on
description: **.NET for Android** (formerly Xamarin.Android) - Open-source Android development bindings for .NET languages. `main` branch targets **.NET 11**.
---

# Instructions for AIs

**.NET for Android** (formerly Xamarin.Android) - Open-source Android development bindings for .NET languages. `main` branch targets **.NET 11**.

## Architecture
- `src/Mono.Android/` - Android SDK bindings in C#
- `src/Xamarin.Android.Build.Tasks/` - MSBuild tasks for Android apps  
- `src/native/` - Native runtime (MonoVM/CoreCLR/NativeAOT)
- `external/Java.Interop/` - JNI bindings and Java-to-.NET interop
- `external/xamarin-android-tools/` - Shared SDK tooling: `AndroidTask`/`AsyncTask` base classes, `AdbRunner`, `EmulatorRunner`, NRT extensions (`IsNullOrEmpty()`, `IsNullOrWhiteSpace()`), `CreateTaskLogger`, and SDK info utilities
- `tests/` - NUnit tests, integration tests, device tests

**Build System:** MSBuild + .NET Arcade SDK + CMake (native)

## Essential Commands
- **Build:** `./build.sh` or `build.cmd`
- **Test with local build:** `dotnet-local.sh`/`dotnet-local.cmd` 
- **Run tests:** `dotnet-local.cmd test bin/TestDebug/net9.0/Xamarin.Android.Build.Tests.dll --filter Name~TestName`
- **Device tests:** `dotnet-local.cmd test bin/TestDebug/MSBuildDeviceIntegration/net9.0/MSBuildDeviceIntegration.dll`

### external/xamarin-android-tools/

The in-tree Android tools libraries include SDK/JDK discovery (`AndroidSdkInfo`, `JdkInfo`, SDK manifest parsing, `AdbRunner`, `EmulatorRunner`) and MSBuild task infrastructure (`AndroidTask`, `AndroidToolTask`, `AsyncTask`, `Files`, `ProcessUtils`, `FileUtil`, `MemoryStreamPool`). They target `netstandard2.0` and/or modern .NET, so verify API availability across all target frameworks before using newer BCL APIs. Prefer `ANDROID_HOME` for new Android SDK environment handling; `ANDROID_SDK_ROOT` is deprecated and should only remain for compatibility.

Useful focused checks:
```sh
dotnet build external/xamarin-android-tools/Xamarin.Android.Tools.sln
dotnet test external/xamarin-android-tools/tests/Xamarin.Android.Tools.AndroidSdk-Tests/Xamarin.Android.Tools.AndroidSdk-Tests.csproj
dotnet test external/xamarin-android-tools/tests/Microsoft.Android.Build.BaseTasks-Tests/Microsoft.Android.Build.BaseTasks-Tests.csproj
```

## Critical Rules

**Never use `git commit --amend`:** Always create new commits. The user will squash or fixup as needed.

Reference official Android documentation where helpful:
* [Android Developer Guide](https://developer.android.com/develop)
* [Android API Reference](https://developer.android.com/reference)
* [Android `aapt2` Documentation](https://developer.android.com/tools/aapt2)

**Only modify the main English `*.resx` files** (e.g., `Resources.resx`)

**Never modify non-English localization files:** `*.lcl` files in `Localize/loc/` or non-English `*.resx` files are auto-generated.

**Use Microsoft docs:** Search MS Learn before making .NET, Windows, or Microsoft features, APIs, or integrations. Use the `microsoft_docs_search` tool.

**MSBuild Tasks:** Extend `AndroidTask` base class, use `XA####` error codes, test in isolation. Use `AsyncTask` for tasks that need `async`/`await` — it handles `Yield()`, `try`/`finally`, and `Reacquire()` automatically.

**Internal build `<UsingTask/>` elements:** For `xa-prep-tasks` and `BootstrapTasks` (internal build-time tasks, not shipped to customers), always use `TaskFactory="TaskHostFactory"` and `Runtime="NET"` attributes on `<UsingTask/>` elements. This runs the task in a separate process to avoid Windows file locking issues and ensures the task runs on .NET (even when MSBuild.exe in Visual Studio uses .NET Framework). Example:

```xml
<UsingTask AssemblyFile="$(BootstrapTasksAssembly)" TaskName="Xamarin.Android.Tools.BootstrapTasks.MyTask" TaskFactory="TaskHostFactory" Runtime="NET" />
<UsingTask AssemblyFile="$(PrepTasksAssembly)" TaskName="Xamarin.Android.BuildTools.PrepTasks.MyTask" TaskFactory="TaskHostFactory" Runtime="NET" />
```

**Do NOT** use `TaskFactory="TaskHostFactory"` or `Runtime="NET"` on `<UsingTask/>` elements shipped in the product (e.g., in `Xamarin.Android.Common.targets` or `Microsoft.Android.Sdk/*.targets`), as it could negatively impact customer builds.

**API Bindings:** Use `[Register]` attributes, follow `Android.*` namespace patterns.

**Native Code:** Use CMake, handle multiple ABIs (arm64-v8a, armeabi-v7a, x86_64, x86).

## Nullable Reference Types

When opting C# code into nullable reference types:

* Only make the following changes when asked to do so.

* Add `#nullable enable` at the top of the file without any preceding blank lines.

* Don't *ever* use `!` (null-forgiving operator) to handle `null`! Always check for null explicitly and throw appropriate exceptions.

* **In test code**, avoid `!` too. Common workarounds:
  - `[SetUp]`-initialized fields: declare as nullable (`MockBuildEngine? engine;`) instead of `MockBuildEngine engine = null!;`
  - After `Assert.IsNotNull`: extract into a local variable (`var opts = task.Options; Assert.IsNotNull (opts); opts.Foo...`) instead of using `task.Options!.Foo`

* Declare variables non-nullable, and check for `null` at entry points.

* Use `throw new ArgumentNullException (nameof (parameter))` in `netstandard2.0` projects.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/android](https://github.com/dotnet/android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
