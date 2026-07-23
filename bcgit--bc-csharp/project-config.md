---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Bouncy Castle for .NET — a cryptography library providing primitives, protocols (CMS, OpenPGP, (D)TLS, TSP, X.509), and NIST PQC algorithms (ML-DSA, ML-KEM, SLH-DSA, Falcon, etc.). This is the **non-FIPS** source tree; the FIPS distribution is separate.

Root namespace: `Org.BouncyCastle`. NuGet package id: `BouncyCastle.Cryptography`.

## Solution layout

Two projects, both under [crypto/](crypto/):

- [crypto/src/BouncyCastle.Crypto.csproj](crypto/src/BouncyCastle.Crypto.csproj) — main library. Multi-targets `net6.0;netstandard2.0;net461`. AOT-compatible on net7+. Signed with [BouncyCastle.NET.snk](BouncyCastle.NET.snk). Versioning is driven by Nerdbank.GitVersioning from [version.json](version.json) — see **API stability and versioning** for the compatibility contract this implies.
- [crypto/test/BouncyCastle.Crypto.Tests.csproj](crypto/test/BouncyCastle.Crypto.Tests.csproj) — NUnit 3 test project. Multi-targets `net6.0;netcoreapp3.1;net472;net461`. Small/inline test data lives in [crypto/test/data/](crypto/test/data/) and is embedded as resources.

**Bulk test data lives in a separate repository.** Clone https://github.com/bcgit/bc-test-data.git — the convention is to put it as a sibling of this repo (`../bc-test-data`), but `SimpleTest.FindTestDataPath` ([SimpleTest.cs](crypto/test/src/util/test/SimpleTest.cs)) walks up from the current working directory until it finds any ancestor containing a folder named `bc-test-data`, so any location on that path works. Tests that depend on it throw `DirectoryNotFoundException` if it is missing.

**Where to put new test vectors / sample files** — no fixed rule. Ask before placing them: propose a location (inline in [crypto/test/data/](crypto/test/data/) for embedded resources vs. the external `bc-test-data` repo) and let the user decide. Size, sensitivity, and whether the vectors are likely to be reused by Java/FIPS sister projects all factor in.

Three build configurations: `Debug`, `Release`, `Publish` (Publish adds deterministic build + assembly signing via [signfile.bat](signfile.bat)).

## Build / test commands

```powershell
# Build the library only
dotnet build crypto/src/BouncyCastle.Crypto.csproj

# Build everything via the solution
dotnet build BouncyCastle.sln

# Run all tests against a specific TFM (CI runs net461, net472, netcoreapp3.1, net6.0)
dotnet test --framework net6.0 crypto/test/BouncyCastle.Crypto.Tests.csproj

# Run a single test fixture
dotnet test --framework net6.0 crypto/test/BouncyCastle.Crypto.Tests.csproj --filter "FullyQualifiedName~ChaCha20Poly1305Test"

# Run a single test method
dotnet test --framework net6.0 crypto/test/BouncyCastle.Crypto.Tests.csproj --filter "FullyQualifiedName=Org.BouncyCastle.Crypto.Tests.ChaCha20Poly1305Test.TestVectors"
```

When writing, modifying, or verifying code that uses hardware intrinsics (anything under [crypto/src/runtime/intrinsics/](crypto/src/runtime/intrinsics/), any `*_X86.cs` file, or code using `System.Runtime.Intrinsics`), see the **intrinsics-testing** skill. It covers the project rule that ISA detection must go through the `Org.BouncyCastle.Runtime.Intrinsics` wrappers (not direct `IsSupported` checks), the planned move toward limited ISA profiles, and the commands to re-run tests with specific instruction sets disabled.

### Mid-session verification expectations

Fast turnaround matters more than exhaustive coverage during an iterative session. Default to:

- Building only the project(s) actually affected by the change (usually `crypto/src/BouncyCastle.Crypto.csproj`, and the test project if tests changed).
- Running just the test fixtures plausibly impacted, via `--filter`, on a single TFM (net6.0 unless the change is TFM-specific).
- Falling back to a wider build/test run only when the change spans broad areas (e.g. core utilities, ASN.1 base types, anything in `crypto/src/util/`).

The final pre-push verification across the full TFM matrix is the user's responsibility — don't block on it.

## Source tree

All source under [crypto/src/](crypto/src/), grouped by domain:

- [crypto/src/crypto/](crypto/src/crypto/) — low-level primitives: `engines/` (block/stream ciphers), `digests/`, `macs/`, `modes/` (CBC, GCM, CCM, …), `paddings/`, `signers/`, `generators/`, `agreement/`, `kems/`, `prng/`, `parameters/`, `tls/`.
- [crypto/src/math/](crypto/src/math/) — `BigInteger`, EC math (`ec/custom/` per-curve specialized impls, `ec/rfc7748/` X25519/X448, `ec/rfc8032/` Ed25519/Ed448), finite-field code, multipliers.
- [crypto/src/asn1/](crypto/src/asn1/) — ASN.1 encoders/decoders + per-standard OID/structure modules (pkcs, x509, cms, …).
- [crypto/src/pqc/](crypto/src/pqc/) — Post-quantum algorithms (`crystals/` for ML-KEM/ML-DSA, plus falcon, bike, cmce, frodo, hqc, ntru, picnic, saber, sphincsplus, lms). These are flagged EXPERIMENTAL in the README.
- [crypto/src/tls/](crypto/src/tls/) — TLS/DTLS protocol implementation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bcgit/bc-csharp](https://github.com/bcgit/bc-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
