---
trigger: always_on
description: This repository is the `cxxmcp` C++ SDK for the Model Context Protocol. Treat it
---

# Agent Instructions

This repository is the `cxxmcp` C++ SDK for the Model Context Protocol. Treat it
as an SDK and package project, not as a standalone MCP server, npm package, CLI,
gateway, or hosted runtime.

## Source Of Truth

- Canonical documentation lives in `README.md`, `README_zh.md`, `docs/`, and
  the generated GitHub Pages site. Do not use GitHub Wiki as an authoritative
  documentation source.
- Release gate details are in `docs/release_gates.md`; package consumption rules
  are in `docs/package_consumption.md`; conformance claims are in
  `docs/conformance_evidence.md`.
- Dependency boundaries and update cadence are summarized in
  `DEPENDENCY_POLICY.md`; detailed release-time dependency review rules are in
  `docs/dependency_policy.md`.
- Keep the repository root small. Prefer adding detailed material under `docs/`
  unless a root file is a standard project file.

## Do Not Touch Casually

- Do not edit generated or local-output paths: `out/`, `build*/`,
  `CMakeFiles/`, `Testing/`, `docs/doxygen/`, `.cache/`, or IDE folders.
- Do not change vendored dependencies under `third_party/` unless the task is an
  explicit dependency upgrade. In `third_party/httplib`, treat `httplib.h`,
  `LICENSE`, and upstream `README.md` as the upstream payload; the minimal
  `CMakeLists.txt` and `README.cxxmcp.md` are cxxmcp-maintained wrapper notes.
- Do not change `.github/dependabot.yml` casually. It is part of the dependency
  update policy, not a generic CI tuning file.
- Do not change `reference/` or RMCP fixture assumptions unless the work is about
  RMCP drift/conformance evidence.
- Do not make SDK public headers expose runtime/gateway/tooling concepts such as
  app/runtime/gateway/CLI/profile/policy/discovery/import/export/trust types, or
  `httplib`. The `sdk_boundary` test enforces this.
- Do not add `FIXME`, `HACK`, or `XXX` markers in first-party source, examples,
  templates, or tests. `scripts/check_source_markers.py` rejects them.

## Public SDK Boundary

- Public SDK headers are under:
  - `sdk/include`
  - `sdk/core/include`
  - `sdk/protocol/include`
  - `sdk/client/include`
  - `sdk/server/include`
  - `sdk/transport/include`
  - `sdk/auth/include` only when auth is enabled
- Public API must remain C++17. Tests and examples may use C++20 where the build
  files already require it.
- If you add or change a public header, update or add a fixture in
  `tests/sdk/public_headers/` so the header compiles independently.
- If you add or rename package targets, update `CMakeLists.txt`,
  `tests/fixtures/package_smoke/`, `tests/CMakeLists.txt`, and package recipes
  together.

## Build Options And Package Contract

- The default project path is SDK-only. Gateway/runtime/CLI tooling belongs in
  external repositories, not in the SDK package surface.
- Keep optional features opt-in:
  - `CXXMCP_ENABLE_HTTP` controls HTTP/SSE transport support.
  - `CXXMCP_ENABLE_AUTH` controls optional OAuth/DPoP auth scaffolding.
  - `CXXMCP_AUTH_CRYPTO=OpenSSL` requires `CXXMCP_ENABLE_AUTH=ON`.
- Default package-manager builds must not pull OpenSSL or full auth into the
  default install path.
- Default source/archive builds use bundled header-only SDK dependencies.
  Registry builds should use `CXXMCP_USE_SYSTEM_DEPS=ON`.
- `cpp-httplib` is a transport implementation detail. Public consumers should
  include cxxmcp HTTP transport headers, not `httplib.h`.

## Protocol And Conformance Work

- Protocol model changes must keep `*_to_json` and `*_from_json` coverage
  symmetric. Run `scripts/check_protocol_model_coverage.py --source .`.
- If a protocol change is based on RMCP sources, update
  `docs/rmcp_source_mapping.json`, `docs/protocol_model_audit.md`, and drift
  evidence together. Run `scripts/check_rmcp_source_drift.py --source .` after
  checking out the pinned RMCP reference if needed.
- Do not update conformance percentages in README/docs unless the underlying
  all-suite conformance evidence has been regenerated.

## Required Local Checks

Run the narrowest check that covers your change. For broad changes, run the full
test preset.

Always run formatting before finishing source changes:

```powershell
pwsh -NoProfile -File scripts\format.ps1 -Check
```

Full SDK test path:

```powershell
cmake --preset tests
cmake --build --preset tests
ctest --preset tests
```

Examples path:

```powershell
cmake --preset examples
cmake --build --preset examples
ctest --preset examples
```

Useful targeted tests:

```powershell
ctest --test-dir out/build/tests -R "^(sdk|client_server|protocol)$" --output-on-failure
ctest --test-dir out/build/tests -R "^(http_transport|stdio_transport|transport_adapters)$" --output-on-failure
ctest --test-dir out/build/tests -R "^(package_smoke|public_targets|sdk_boundary)$" --output-on-failure
```

Source/release consistency checks:

```powershell
python scripts/check_protocol_model_coverage.py --source .
python scripts/check_sdk_header_boundaries.py --source .
python scripts/check_source_markers.py --source .
python scripts/check_package_auth_features.py --source .
python scripts/check_package_recipe_sync.py --source .
python scripts/check_p2_todo_status.py --source .
```

## Change-Specific Expectations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caomengxuan666/cxxmcp](https://github.com/caomengxuan666/cxxmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
