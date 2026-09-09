---
trigger: always_on
description: The root solution, `ReSharper.Structured.Logging.slnx`, groups the .NET plugin, tests, build automation, and documentation. Core ReSharper and Rider backend code lives in `src/ReSharper.Structured.Logging/`, organized by responsibility (`Analyzer/`, `Highlighting/`, `QuickFixes/`, `Settings/`, and related support folders). Rider frontend Kotlin and resources are under `src/rider/main/`. NUnit fixtures are in `test/src/`; analyzer and quick-fix inputs plus expected `.gold` output files are in `te
---

# Repository Guidelines

## Project Structure & Module Organization

The root solution, `ReSharper.Structured.Logging.slnx`, groups the .NET plugin, tests, build automation, and documentation. Core ReSharper and Rider backend code lives in `src/ReSharper.Structured.Logging/`, organized by responsibility (`Analyzer/`, `Highlighting/`, `QuickFixes/`, `Settings/`, and related support folders). Rider frontend Kotlin and resources are under `src/rider/main/`. NUnit fixtures are in `test/src/`; analyzer and quick-fix inputs plus expected `.gold` output files are in `test/data/`. Rule documentation belongs in `rules/`, screenshots in `images/`, and NUKE orchestration in `build/Build.cs`.

## Build, Test, and Development Commands

Run commands from the repository root:

- `build.cmd Compile` (Windows) or `./build.sh Compile` (Unix): restore and compile the ReSharper plugin.
- `build.cmd Test`: build the test project and run it with NUnit Console.
- `build.cmd Pack`: compile and create the ReSharper NuGet package; this is the default NUKE target.
- `build.cmd PackRiderPlugin --is-rider-host`: build and package the Rider plugin through Gradle.
- `build.cmd RunIde --is-rider-host [--run-ide-solution <path>]`: launch a sandboxed Rider with the plugin installed for manual testing; the optional solution path is opened on start, for example `--run-ide-solution test/manual/Issue130/Issue130.slnx`.
- `build.cmd UpdateSdkVersion [--sdk-version-override <version>]`: adopt a newer JetBrains SDK in `Directory.Build.props`; see "Adopting a new SDK".
- `dotnet sln ReSharper.Structured.Logging.slnx list`: verify that solution project links resolve.

The build requires a compatible .NET SDK; Rider packaging also requires a JDK 17 or newer to run Gradle. The JDK that the Rider build itself compiles against is derived from the target Rider version and provisioned automatically by the Foojay toolchain resolver configured in `settings.gradle`, so it does not need to be installed by hand. Generated output appears in `bin/`, `gradle-build/`, and repository-root package files and must not be committed.

## Releasing

Releases are published from the `Build` workflow, not from a tag. It publishes when the `publish` input is enabled on a manual run (`gh workflow run build.yml --ref master -f publish=true`), and also when a push to `master` changes `SdkVersion` in `Directory.Build.props`, which is how an SDK update ships itself. Either way it packs both plugins, pushes the ReSharper `.nupkg` and the Rider `.zip` to JetBrains Marketplace, then creates the git tag and GitHub release. Publishing needs the `JETBRAINS_MARKETPLACE_TOKEN` repository secret, a permanent token from <https://plugins.jetbrains.com/author/me/tokens>.

The published version and the tag are `<SdkVersion>.<workflow run number>`, where `SdkVersion` comes from `Directory.Build.props`. Marketplace rejects a version it already has, so a failed publish must be re-dispatched rather than re-run, and Marketplace moderation means a successful run only says the update was uploaded. An EAP `SdkVersion` suffix routes the Rider plugin to the `eap` channel and marks the GitHub release as a prerelease.

The corresponding NUKE targets are `PublishReSharperPlugin` and `PublishRiderPlugin --is-rider-host`; both read the token from the `MARKETPLACE_TOKEN` environment variable and refuse to run without it.

## Adopting a new SDK

The `SDK update` workflow polls nuget.org daily and proposes the bump itself. `build.cmd UpdateSdkVersion` is what it runs: the target reads the versions published for all four SDK packages, keeps only those every one of them has, and picks a target under the wave policy. While the adopted version is stable only a higher wave qualifies, because a same-wave patch is already covered by the `Wave` dependency range the package declares; once it is a prerelease the whole train is followed, `eap01` through `rc01` to the stable release that closes the wave. `--sdk-version-override <version>` adopts a specific version instead, which is the way to take a same-wave patch.

The workflow then commits the bump to `sdk-update/<version>`, opens a pull request with auto-merge enabled, and lets `Build and test` decide. Green merges to `master`, which publishes; red leaves the pull request open, which is the normal outcome for a wave change. Expect to fix binding redirects in `test/src/app.config`, `.gold` expectations, SDK API breaks, and sometimes `build.gradle` and the Gradle wrapper. A stale red pull request is closed as superseded when the next version comes along.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olsh/resharper-structured-logging](https://github.com/olsh/resharper-structured-logging) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
