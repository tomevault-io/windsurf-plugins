---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Keep this CLAUDE.md file up to date.** When the user tells you something about the project (conventions, patterns, warnings), eagerly add it here. If you make changes that conflict with what's documented here, update this file to match. The user will use git to revert if they disagree.

## Project Overview

Velopack is an installation and auto-update framework for cross-platform desktop applications. It has a C#/.NET frontend (library + CLI tool called `vpk`) and a Rust backend (update binaries for Windows/macOS/Linux). The library supports C#, C++, Node.js, Python, and Rust clients.

## CRITICAL: Do Not `cd` to the Repo Root

**The working directory is ALREADY the repo root.** Never write `cd /c/Source/velopack &&` or any variation — you are already there. Just run commands directly (e.g. `cargo build`, `dotnet test test/Velopack.Tests`). You may `cd` into a subdirectory if a specific task genuinely requires it, but never `cd` to the repo root itself.

## Build Commands

```bash
# .NET (main solution)
dotnet build                          # Build all .NET projects (Debug)
dotnet build -c Release               # Build all .NET projects (Release)

# Rust
cargo build                           # Build all Rust workspace members (Debug)
cargo build --release                  # Build all Rust workspace members (Release)
cargo bw                              # Alias for `cargo build --features windows` — required on Windows
                                      # to also build stub.exe/setup.exe (needed by packaging tests)
```

## Test Commands

```bash
# Run all .NET tests
dotnet test

# Run a specific test project (must use --project)
dotnet test --project test/Velopack.Tests
dotnet test --project test/Velopack.CommandLine.Tests
# Velopack.Packaging.Tests runs in well under a minute since TestApp publishes are
# cached per process and collections parallelize; running the whole project is fine.
# Targeted subsets: tests use xunit v3 on Microsoft.Testing.Platform, so filters go
# after `--` and use --filter-class / --filter-method with * wildcards
# (NOT the old VSTest --filter "FullyQualifiedName~..." syntax):
dotnet test --project test/Velopack.Packaging.Tests -- --filter-class "*MsiTests"

# Run a single test by name
dotnet test --project test/Velopack.Tests -- --filter-method "*TestMethodName*"

# Rust tests
cargo test
```

**Run tests locally in Debug (the default), never `-c Release`.** In Debug, `HelperFile` and
`PathHelper.GetRustBuildOutputDir()` resolve unsuffixed rust binaries from `target/debug`, so a plain
`cargo build` (add `--features windows` on Windows) is all the setup needed. Release-mode tests
expect CI's arch-suffixed vendored binaries (`update_x64.exe` etc.) — do not fake those locally by
copying files into `target/release`.

## Repository Structure

```
src/
├── lib-csharp/          # Core C# library (Velopack NuGet package)
│   ├── Sources/         # Update sources (Velopack Flow, GitHub, GitLab, Gitea, HTTP, file)
│   ├── Locators/        # Platform-specific app locators (Windows, Linux, OSX)
│   ├── NuGet/           # Package handling (ZipPackage, PackageManifest)
│   └── UpdateManager.cs, VelopackApp.cs  # Primary public API
├── lib-rust/            # Core Rust library
├── bins/                # Rust update binaries (update.exe, UpdateNix, UpdateMac)
├── lib-cpp/             # C++ bindings via cbindgen
├── lib-nodejs/          # Node.js FFI bindings (neon)
├── lib-python/          # Python bindings via PyO3
├── vpk/                 # CLI tool entry point
│   ├── Velopack.Core/           # Core CLI abstractions
│   ├── Velopack.Packaging/      # Base packaging logic
│   ├── Velopack.Packaging.Windows/  # MSI, setup.exe (Handlebars templates)
│   ├── Velopack.Packaging.Unix/     # AppImage, tarball
│   ├── Velopack.Deployment/     # Upload to GitHub, S3, Azure, etc.
│   ├── Velopack.Flow/           # Application flow/orchestration
│   └── Velopack.Vpk/           # CLI entry using System.CommandLine
├── code-generator/      # Cross-language type generator
└── wix-dll/             # WiX MSI integration
test/
├── Velopack.Tests/              # Core library unit tests
├── Velopack.Packaging.Tests/    # Packaging/CLI tests
├── Velopack.CommandLine.Tests/  # CLI command parsing tests
├── Velopack.Deployment.Tests/   # vpk destination + cross-language update-source tests (docker stack)
├── TestApp/                     # Test application used by integration tests
├── fixtures/                    # Test fixture files
├── PathHelper.cs                # Shared test utilities (linked into all test projects)
└── GlobalUsings.cs              # Shared usings (linked into all test projects)
samples/                         # Example apps (C#, C++, Node.js, Python, Rust)
```

## Architecture

**Core libraries**: `lib-csharp` and `lib-rust` are the two core libraries and must be kept in sync. Other language bindings (Node.js, Python, C++) are built on top of `lib-rust`. The libraries invoke the update binaries (`src/bins/` — e.g. `update.exe`) to perform actual install/update operations.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velopack/velopack](https://github.com/velopack/velopack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
