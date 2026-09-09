---
trigger: always_on
description: A Flutter plugin published to pub.dev: an encrypted key/value store, optionally
---

# biometric_storage

A Flutter plugin published to pub.dev: an encrypted key/value store, optionally
gated behind a biometric prompt. Android (KeyStore), iOS and macOS (Keychain +
LocalAuthentication), Linux (libsecret), Windows (wincred), Web (localStorage,
unencrypted — say so whenever it comes up).

The Dart surface is one class, `BiometricStorage`, in
[`lib/src/biometric_storage.dart`](lib/src/biometric_storage.dart). Everything
else is a platform implementation behind it.

## The one thing that is not obvious

**The Windows implementation is compiled on every platform that has
`dart.library.io`** — iOS, macOS, Android and Linux included. Two things put it
there, and they are not independent:

1. `lib/biometric_storage.dart` re-exports `src/biometric_storage_win32.dart`
   under `if (dart.library.io)`.
2. Flutter generates `.dart_tool/flutter_build/dart_plugin_registrant.dart` for
   *all* platforms at once, not per build target. An iOS release build's copy
   contains `import 'package:biometric_storage/biometric_storage.dart'` and a
   `Platform.isWindows` branch calling
   `Win32BiometricStoragePlugin.registerWith()`.

So a breaking change in `package:win32` breaks an **iOS** build. That is how
5.x ended up unusable: `win32` 6 removed `TEXT()`, and the error surfaced as a
failing `flutter test` on macOS.

Dropping the export in (1) is not an escape: the registrant needs
`Win32BiometricStoragePlugin` to be *on* the barrel, so removing it breaks the
compile everywhere instead. Naming the file with `dartFileName:` would make the
registrant import `src/biometric_storage_win32.dart` directly — on every
platform, so nothing is gained. Nor does splitting Windows out into a federated
`biometric_storage_windows` package: the registrant would import that package
instead, on every platform, exactly as before.

The defence is therefore a compile, not a restructuring.
`test/biometric_storage_test.dart` imports the public barrel rather than `src/`
precisely so that `flutter test` on any host compiles the win32 bindings, and CI
runs the suite on Windows too. Do not "tidy" that import.

Related: the `windows:` block in `pubspec.yaml` has no `dartFileName`, so the
registrant imports the barrel. `fileName:` is a **web-only** key — it sat under
`windows:` for years and was silently ignored.

## Layout

* `darwin/biometric_storage/Sources/biometric_storage/` — the iOS and macOS
  Swift sources, shared via `sharedDarwinSource: true` in `pubspec.yaml`.
  `BiometricStorageImpl.swift` is platform-agnostic; `BiometricStoragePlugin.swift`
  has the `#if os(iOS)` / `#if os(macOS)` split (the registrar's messenger is a
  method on iOS and a property on macOS).
* `darwin/biometric_storage/Package.swift` **and** `darwin/biometric_storage.podspec`
  describe the same sources. Both must be kept in step — CocoaPods stays
  supported until the registry goes read-only, and Flutter picks whichever the
  consuming app uses. `Package.swift` must keep its `FlutterFramework`
  dependency; without it Flutter warns on every build.
* `android/` — Kotlin, `design.codeux.biometric_storage`.
* `lib/src/biometric_storage_web.dart`, `linux/`, `lib/src/biometric_storage_win32.dart`.
* `example/` — the app the CI builds. Its own `flutter pub get` runs from
  `example/`, not the repository root.

## Commands

The whole test suite. Fast, and it is what catches a `package:win32` break.

```bash
flutter test
```

`--fatal-infos` is the point: without it the promotions in
`analysis_options.yaml` change nothing at the command line. It covers only the
package you stand in, so run it in `example/` too when that is what changed.

```bash
flutter analyze --fatal-infos
```

Run before committing.

```bash
dart format lib test example/lib
```

## Verifying

**A green analyze here is an answer about one resolution, not about consumers'.**
`flutter analyze` does flag an undefined win32 symbol in this package's own
sources — but it flags it against whatever `pubspec.yaml` resolves to, so while
the constraint said `win32 <6.0.0` it stayed green for a break that every app
resolving win32 6 would hit. What a version bump is actually verified by is a
build that finished, in an app that has the dependency in question.

**Gradle and Xcode are never covered by an analyze at all.** A toolchain or
androidx bump is proven by `flutter build` in `example/`, nothing less.

**Ask what would make this test red.** Green is information only if failure was
reachable. When you change something the win32 guard is supposed to catch,
reintroduce the break once and watch it fail before trusting the pass.

**A zero exit code is not evidence an edit landed.** Grep for the new state.

**Prove a plugin actually linked, rather than that the build was quiet.** A
plugin that fails to register produces a perfectly successful build. `nm` the
built binary for the plugin's symbols, or read the generated
`GeneratedPluginRegistrant.swift`.

**Resolution is proven in a consumer app, not here.** The interesting failures
are version conflicts with packages this repo does not depend on. Generate a
throwaway app in the scratchpad, point it at this checkout by path, add the
package that conflicts, and run `flutter pub get`.

## Android


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [authpass/biometric_storage](https://github.com/authpass/biometric_storage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
