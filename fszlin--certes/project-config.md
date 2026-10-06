---
trigger: always_on
description: Certes is an embeddable .NET ACME client library and a companion CLI. The project
---

# Development guide for coding agents

## Purpose and scope

Certes is an embeddable .NET ACME client library and a companion CLI. The project
is being revived incrementally. Preserve working behavior and compatibility
while making builds, tests, dependencies, and certificate workflows maintainable.

This file applies throughout the repository. Read it and the relevant code/tests
before editing. Treat roadmap items as context, not authorization to expand the
current task. Keep this guide accurate when commands, targets, or blockers change.

## Project map

| Path | Responsibility |
| --- | --- |
| `src/Certes/AcmeContext.cs`, `IAcmeContext.cs` | Public entry point and account/order operations |
| `src/Certes/Acme/` | HTTP transport, resource contexts, protocol models, error handling |
| `src/Certes/Extensions/` | Convenience APIs, order generation, certificate export |
| `src/Certes/Jws/`, `Crypto/`, `Json/` | Signing, key algorithms, wire serialization |
| `src/Certes/Pkcs/` | CSR, certificate-chain, and PFX utilities |
| `src/Certes.Cli/` | Commands, dependency injection, and user settings |
| `test/Certes.Tests/` | Offline xUnit/Moq tests and ephemeral certificate fixtures |
| `test/Certes.Tests.Integration/` | ACME integration flows using local Pebble and challtestsrv |
| `misc/certes.props` | Shared build settings, signing, versions, warnings-as-errors |
| `docs/` | Usage/API documentation and DocFX site |
| `docs/releasing.md` | Release setup, tagging flow, and publish recovery runbook |
| `.github/workflows/build.yml` | Cross-platform compilation and unit tests, package smoke checks |
| `.github/workflows/codeql.yml` | C# and GitHub Actions static security analysis |
| `scripts/PackageSmoke/` | Consumer of the locally packed library; not a solution project |
| `scripts/StagingVerification/` | Opt-in published-package Let's Encrypt staging check; requires Cloudflare DNS credentials |
| `scripts/Coverage/` | .NET 10/Linux unit coverage settings and local commands |
| `scripts/Pebble/` | Pinned container stack, readiness probe, and local integration instructions |
| `.github/workflows/release.yml` | Tag-driven package release workflow |

`README.md` is the repository landing page. `docs/README.md` is the detailed usage
guide and is included by `docs/index.md` in the documentation site. Update the
appropriate guide when behavior changes; avoid duplicating long usage examples.

## Working approach

- Inspect the working tree first; preserve unrelated or user-authored changes.
- Keep each change focused. Separate dependency migrations, behavior fixes, and
  broad formatting changes so their effects can be verified independently.
- Follow `.editorconfig` and nearby code: four-space C# indentation, Allman
  braces, existing namespaces and naming. Do not mass-reformat files.
- Prefer existing abstractions and test helpers. Add dependencies only for a
  concrete need; keep provider-specific dependencies out of the core library.
- Keep core-library runtime dependency minimums at the lowest supported, secure
  versions that provide required behavior. Raise them for security fixes or a
  concrete compatibility/feature need, not simply because newer versions exist.
  Review consumer impact separately from CLI, test, and build-tool updates.
  Dependabot skips routine SemVer updates for the core runtime packages while
  preserving security updates; review those dependency floors deliberately.
  Routine major-version updates for NuGet and GitHub Actions are also ignored;
  plan those migrations explicitly. Weekly patch/minor updates remain enabled
  outside the core runtime exclusions. SemVer ignore rules do not suppress
  security updates. Pre-1.0 minor releases can still break APIs; routine Spectre
  updates remain excluded and require deliberate review after the v5 migration (#405).
- Public API changes must account for interfaces, extension methods, downstream
  implementers, serialization, and all retained target frameworks. Explain
  intended breaking changes rather than introducing them incidentally.
- Preserve assembly signing and package identity unless changing them is part
  of the task. `SkipSigning=true` is a local/test aid, not a release setting.
- Do not hand-edit generated `Strings.Designer.cs` files, build outputs, or
  generated API documentation. Change the source resource/generator input.
  `Strings.Designer.cs` is regenerated at design time only (Visual Studio, or
  MSBuild's `GenerateResource` task with the `StronglyTyped*` parameters);
  `StringsTests.EveryDesignerMemberHasAResource` fails if the CLI copy is stale.
- Add focused regression tests for behavior fixes. Assert observable behavior,
  especially failure paths, rather than duplicating implementation details.
- Report files changed, checks actually run, outcomes, and blockers. Distinguish
  compilation success, test success, and verified CA interoperability.

## Branching and releases

- v4.1.0 is released. v4.x is maintenance-only for security fixes and reported
  regressions. The 5.0.0 milestone tracks cancellation (#299) and the coordinated
  Spectre CLI migration (#405); Native AOT (#313) remains unscheduled.
- Use short-lived branches for focused tasks, based on current `main`. Examples:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fszlin/certes](https://github.com/fszlin/certes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
