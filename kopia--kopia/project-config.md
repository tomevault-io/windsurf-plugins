---
trigger: always_on
description: - Check for hardcoded secrets, API keys, or credentials
---

# Kopia Copilot Instructions

## When reviewing code, focus on:

### Security Critical Issues
- Check for hardcoded secrets, API keys, or credentials
- Verify proper input validation and sanitization
- Review authentication and authorization logic

### Code Quality Essentials
- Functions should be focused and appropriately sized
- Use clear, descriptive naming conventions
- Ensure proper error handling throughout

### Performance Issues
- Spot inefficient loops and algorithmic issues
- Check for memory leaks and resource cleanup
- Review caching opportunities for expensive operations

## Review Style
- Be specific and actionable in feedback
- Explain the rationale behind recommendations
- Acknowledge good patterns when you see them
- Ask clarifying questions when code intent is unclear

## Review Test Coverage
- Ensure there are tests that cover and exercise the new or changed functionality

Always prioritize security vulnerabilities and performance issues that could impact users.

Always suggest changes to improve readability.

## Repository Overview

Kopia is a fast and secure open-source backup/restore tool written in **Go** that creates encrypted snapshots and saves them to remote  storage. The repository is approximately 15MB with ~1,000 Go files.

**Key Technologies:**
- **Backend:** Go (primary language)
- **Build System:** GNU Make with cross-platform support (Windows/Linux/macOS/ARM)
- **UI:** React-based HTML UI (embedded via go:embed, source at github.com/kopia/htmlui)
- **Desktop App:** Electron-based KopiaUI wrapper
- **Website:** Hugo static site generator

## Build Commands

### Setup (Required Before Building)
```bash
make -j4 ci-setup
```
**Time:** ~30-60 seconds
**What it does:** Downloads Go modules, installs build tools (gotestsum, golangci-lint, hugo, node), and installs npm dependencies for the app. **ALWAYS run this after cloning or when build tools are missing.**

## Linting

**Run linter:**
```bash
make lint
```
**Time:** ~3-4 minutes
**Linter:** golangci-lint with timeout of 1200s
**Config:** `.golangci.yml` (extensive configuration with 40+ enabled linters)

**Auto-fix linting issues:**
```bash
make lint-fix
```

Note: Linting is **NOT** run on linux/arm64 or linux/arm platforms to avoid issues.

**Check code locks:**
```bash
make check-locks
```
**Note:** Not available on linux/arm64 or linux/arm.


### Building Kopia CLI

**Build without UI (faster for testing):**
```bash
make install-noui
```
**Output:** `~/go/bin/kopia`
**Time:** ~5-10 seconds
**Use this for:** Testing CLI changes that don't involve the UI.

**Race detector build:**
```bash
make install-race
```
**Use this for:** Debugging race conditions.

**Full build with embedded HTML UI:**
```bash
make install
```
**Output:** `~/go/bin/kopia`
**Time:** ~10-20 seconds
**Note:** Embeds HTML UI from github.com/kopia/htmluibuild dependency.

### Building KopiaUI Desktop App

**Prerequisites:** Must build kopia CLI first (creates embedded binary).

```bash
make kopia-ui
```
**Output:** `dist/kopia-ui/` directory with platform-specific installers
**Time:** ~2-5 minutes
**Note:** Only works on amd64 architectures. On Linux, may require xvfb for headless testing.

## Testing

### Unit Tests (Standard)
```bash
make test
```
**Time:** ~2-4 minutes
**Runs:** All unit tests with gotestsum
**Timeout:** 1200s (20 minutes) per test
**Format:** pkgname-and-test-fails

### Unit Tests with Coverage
```bash
make test-with-coverage
```
**Output:** `coverage.txt`
**Time:** ~3-5 minutes
**Note:** Used by Code Coverage workflow. Sets KOPIA_COVERAGE_TEST=1

### Integration Tests
```bash
make build-integration-test-binary  # Build test binary first
make integration-tests
```
**Time:** ~5-10 minutes
**Requires:** KOPIA_INTEGRATION_EXE environment variable

**Race Detector Tests:**
```bash
make test UNIT_TEST_RACE_FLAGS=-race UNIT_TESTS_TIMEOUT=1200s
```

### CI Test Suites
```bash
make ci-tests  # Runs: vet + test
make ci-integration-tests  # Runs: robustness-tool-tests + socket-activation-tests
```

### Provider Tests (Cloud Storage)
```bash
make provider-tests PROVIDER_TEST_TARGET=...
```
**Time:** 15 minutes timeout
**Requires:** KOPIA_PROVIDER_TEST=true, credentials for storage backend.
**Note:** Tests various cloud storage providers (S3, Azure, GCS, etc.)

### Other Test Types
- `make compat-tests` - Compatibility tests with older Kopia versions
- `make endurance-tests` - Long-running endurance tests (1 hour timeout)
- `make robustness-tests` - Robustness testing with FIO
- `make stress-test` - Stress tests (1 hour timeout)
- `make htmlui-e2e-test` - HTML UI end-to-end tests (10 minutes timeout)

## Common Issues & Workarounds

### Build Issues

1. **Missing build tools error:** Always run `make -j4 ci-setup` first after cloning.

2. **Go version mismatch:** Kopia requires the Go toolchain with the version specified in go.mod. The `go-version-file` is used in GitHub Actions.

3. **Platform-specific builds:**
   - macOS: Creates universal binaries (AMD64 + ARM64) with `lipo`
   - Windows: Requires chocolatey packages: make, zip, unzip, curl
   - Linux ARM: Uses goreleaser for multi-arch builds on AMD64 host

4. **KopiaUI build failures on ARM:** KopiaUI (Electron app) only builds on amd64. The build is skipped on ARM architectures.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kopia/kopia](https://github.com/kopia/kopia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
