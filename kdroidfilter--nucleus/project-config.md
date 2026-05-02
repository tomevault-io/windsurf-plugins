---
trigger: always_on
description: A multi-module Gradle plugin and runtime library toolkit for shipping production-ready JVM desktop applications on macOS, Windows, and Linux.
---

# ComposeDeskKit (Nucleus)

A multi-module Gradle plugin and runtime library toolkit for shipping production-ready JVM desktop applications on macOS, Windows, and Linux.

## Project Structure

- `core-runtime` - Executable type detection, single instance, deep links, platform detection, app metadata (`NucleusApp`)
- `aot-runtime` - AOT cache mode detection for JDK 25+ (Project Leyden)
- `updater-runtime` - Auto-update engine (GitHub/S3), SHA-512 verification, progress tracking, update level detection, post-update events
- `freedesktop-icons` - Type-safe freedesktop Icon Naming Specification constants (shared by notification-linux and launcher-linux)
- `notification-linux` - Freedesktop Desktop Notifications API via JNI (D-Bus org.freedesktop.Notifications)
- `notification-windows` - Windows Toast Notifications API via JNI (WinRT)
- `launcher-windows` - Windows Launcher API via JNI (WinRT/COM) — badge notifications, jump lists (ICustomDestinationList), overlay icons, and thumbnail toolbar buttons (ITaskbarList3) on taskbar
- `launcher-linux` - Unity Launcher API via JNI (badge, progress, urgency, quicklist via com.canonical.Unity.LauncherEntry + com.canonical.dbusmenu)
- `taskbar-progress` - Native taskbar/dock progress bar and attention requests (Windows ITaskbarList3, macOS NSDockTile, Linux delegates to launcher-linux)
- `darkmode-detector` - Reactive OS dark mode detection via JNI
- `system-color` - Reactive system accent color and high contrast detection via JNI
- `energy-manager` - Energy efficiency & screen-awake APIs
- `native-ssl` / `native-http` / `native-http-okhttp` / `native-http-ktor` - OS trust store integration
- `linux-hidpi` - Native HiDPI scale detection on Linux
- `graalvm-runtime` - GraalVM native-image bootstrap
- `decorated-window-core` - Shared types, layout, styling (design-system agnostic)
- `decorated-window-jbr` - JBR-based implementation (requires JetBrains Runtime)
- `decorated-window-jni` - JNI-based implementation (any JVM, GraalVM compatible)
- `decorated-window-jewel` - Jewel (IntelliJ theme) integration
- `decorated-window-material2` - Material 2 color mapping
- `decorated-window-material3` - Material 3 color mapping
- `plugin-build/plugin` - Gradle plugin for packaging & distribution
- `example` / `jewel-sample` / `sample-cmp` - Sample applications

## Build & Run

```bash
./gradlew run                                    # Run example app
./gradlew packageDistributionForCurrentOS        # Package for current OS
./gradlew packageReleaseDistributionForCurrentOS # Release build with ProGuard
./gradlew preMerge                               # Full CI verification
./gradlew reformatAll                            # Format all code
```

## Key Technologies

- Kotlin 2.3+ with Compose Desktop 1.10+
- JNI for all native interop (no JNA in runtime modules)
- JBR (JetBrains Runtime) API for decorated-window-jbr
- Gradle 9+ with version catalog (`gradle/libs.versions.toml`)
- Detekt + KtLint for code quality

## Development Notes

- Target: JDK 17+ runtime, JDK 25+ recommended for AOT
- JNI code: be careful with macOS ARC/retain and weak references
- Native modules use platform-specific JNI implementations — test on each OS
- Plugin is published via included build in `plugin-build/`
- Version catalog is the source of truth for all dependency versions
- `decorated-window-jni` is the recommended backend for new projects (true Windows fullscreen, GraalVM compatible)
- macOS Liquid Glass enabled by default via `macOsSdkVersion = "26.0"` (vtool SDK patching)

## Adding a Native JNI Module

When creating a new module with platform-specific JNI libraries, all steps below are required:

1. **Native source** — `<module>/src/main/native/{linux,macos,windows}/` with `build.sh`/`build.bat` + C/ObjC source. Library name: `nucleus_<feature>`. Linux: prefer `dlopen` over hard compile-time deps.
2. **Build output** — scripts must place binaries in `<module>/src/main/resources/nucleus/native/{linux-x64,linux-aarch64,darwin-x64,darwin-aarch64,win32-x64,win32-aarch64}/`. Build scripts must also clear the `NativeLibraryLoader` cache (`~/.cache/nucleus/native/<arch>/`) after compilation, otherwise the loader serves the stale cached copy instead of the freshly built library.
3. **Kotlin JNI bridge** — `internal object` using `NativeLibraryLoader.load()` with `@JvmStatic external` methods. Always provide a Kotlin fallback when native lib is unavailable.
4. **GraalVM reachability metadata** — create `<module>/src/main/resources/META-INF/native-image/io.github.kdroidfilter/nucleus.<module>/reachability-metadata.json` declaring all JNI-accessible classes/methods. Without this, native-image silently eliminates the bridge.
5. **CI build** (`build-natives.yaml`) — add build + verify + upload steps for each platform (Windows, macOS, Linux matrix x64+aarch64). Artifact naming: `<module>-<platform>`.
6. **CI consumers** — add download step + EXPECTED verify entries (all 6 arch paths) in **every** consumer workflow: `pre-merge.yaml`, `publish-maven.yaml`, `publish-plugin.yaml`, `test-packaging.yaml`, `test-graalvm.yaml`, `release-graalvm.yaml`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdroidFilter/Nucleus](https://github.com/kdroidFilter/Nucleus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
