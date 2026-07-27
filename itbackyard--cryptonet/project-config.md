---
trigger: always_on
description: CryptoNet is a cross-platform .NET cryptography library providing RSA, AES, and DSA implementations. The core library targets .NET Standard 2.0 for maximum compatibility, while tests and examples use .NET 8.0.
---

# CryptoNet - AI Coding Agent Instructions

## Project Overview
CryptoNet is a cross-platform .NET cryptography library providing RSA, AES, and DSA implementations. The core library targets .NET Standard 2.0 for maximum compatibility, while tests and examples use .NET 8.0.

## Architecture

### Project Structure
- **CryptoNet** (netstandard2.0): Core library with `CryptoNetRsa`, `CryptoNetAes`, `CryptoNetDsa` classes
- **CryptoNet.Models** (netstandard2.0): Shared models (`CryptoNetInfo`, `RsaDetail`, `AesDetail`, `DsaDetail`, `KeyType`, `EncryptionType`)
- **CryptoNet.ExtShared** (netstandard2.0): Internal utilities for certificate handling and byte operations
- **CryptoNet.ExtPack** (net8.0): Extension methods for MD5 hashing, PEM export, and content validation
- **CryptoNet.UnitTests** (net8.0): NUnit tests with NUnit, Moq, Shouldly assertions

### Key Dependencies
All projects use **Central Package Management** via `Directory.Packages.props`. Never add package versions directly to `.csproj` files—only add `<PackageReference Include="PackageName" />` without versions.

### NuGet Packaging Pattern
The main `CryptoNet.csproj` uses `PrivateAssets="all"` for project references to `ExtShared` and `Models`, then explicitly includes their DLLs in the package via `IncludeReferencedDllsInPackage` target. This creates a self-contained NuGet package.

## Development Workflows

### Building & Testing
```powershell
# Standard build and test
dotnet build
dotnet test --configuration Release --no-build

# Code coverage with HTML report
.\Scripts\run_codecoverage.ps1  # Runs tests, generates coverage, opens in Edge

# Docker build
.\Scripts\run_docker_build.ps1
# or: docker build . --file .\Dockerfile --tag cryptonet-service:latest
```

### Documentation Generation
Documentation uses **DocFX** to generate API docs from XML comments:
```powershell
.\Scripts\run_docs.ps1  # Cleans, builds, serves on localhost:8080

# Setup (one-time): dotnet tool install -g docfx
```

The `index.md` is kept in sync with `README.md`:
```powershell
.\Scripts\run_update_index.ps1  # Adds YAML header and appends README content
```

### Release Process
Releases are tag-based with version format `vX.Y.Z` or `vX.Y.Z-previewYYYYMMDDHmm`:
```powershell
# Preview release
.\Scripts\run_release.ps1 -VersionNumber 3.0.0 -IsPreview $true

# Production release
.\Scripts\run_release.ps1 -VersionNumber 3.0.0 -IsPreview $false
```

Pushing tags triggers GitHub Actions workflows (`cd-release.yml`, `cd-release-preview.yml`) that build, test, pack, and publish to NuGet.

## Coding Conventions

### Code Quality Standards
- **Nullable reference types enabled**: Use `?` for nullable types, handle nulls explicitly
- **TreatWarningsAsErrors**: All warnings are errors—code must be warning-free
- **XML documentation required**: Public APIs must have `<summary>` tags (enforced by `GenerateDocumentationFile`)
- **Deterministic builds**: `<Deterministic>true</Deterministic>` in `Directory.Build.Props`

### Interface Pattern
All main crypto classes implement interfaces (`ICryptoNetRsa`, `ICryptoNetAes`, `ICryptoNetDsa`) which inherit from `ICryptoNet`. This base interface defines:
```csharp
CryptoNetInfo Info { get; }
byte[] EncryptFromString(string content);
string DecryptToString(byte[] bytes);
byte[] EncryptFromBytes(byte[] bytes);
byte[] DecryptToBytes(byte[] bytes);
```

### Constructor Overloading Pattern
Crypto classes follow a consistent constructor pattern:
```csharp
// Self-generated keys
public CryptoNetRsa(int keySize = 2048)

// String key (XML or PEM)
public CryptoNetRsa(string key, int keySize = 2048)

// File-based key
public CryptoNetRsa(FileInfo fileInfo, int keySize = 2048)

// X509 certificate
public CryptoNetRsa(X509Certificate2? certificate, KeyType keyType, int keySize = 2048)
```

### Testing Patterns
Tests use **NUnit** with **Shouldly** for assertions:
```csharp
result.ShouldBe(expected);  // NOT Assert.AreEqual
ExtensionPack.CheckContent(original, decrypted).ShouldBeTrue();  // MD5-based content comparison
```

Test resource files are in `Resources/` with `<CopyToOutputDirectory>Always</CopyToOutputDirectory>`.

## Common Pitfalls

### Version Management
- **Never** hardcode versions in individual `.csproj` files
- Update versions in `Directory.Build.Props` (global) and `Directory.Packages.props` (package versions)
- Release versions must match the format `^\d+\.\d+\.\d+$` in `run_release.ps1`

### Cross-Platform Compatibility
- Core library must remain .NET Standard 2.0 compatible
- Avoid .NET 8+ specific APIs in `CryptoNet`, `CryptoNet.Models`, `CryptoNet.ExtShared`
- Use `System.Security.Cryptography` types from .NET Standard 2.0

### GitHub Actions Workflows
- **Never add `runs-on` to caller jobs** when using reusable workflows (see `CONTRIBUTING.md`)
- Reusable workflows declare `on: workflow_call` and define their own `runs-on`
- Example: `cd-build-test-pack.yml` is a reusable workflow called by release workflows

### File Header Conventions
All C# files include copyright headers:
```csharp
// <copyright file="FileName.cs" company="itbackyard" year="2021">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itbackyard/CryptoNet](https://github.com/itbackyard/CryptoNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
