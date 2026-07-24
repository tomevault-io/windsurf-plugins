---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Serverpod Development Guide

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

Serverpod is a next-generation app and web server framework built for the Flutter community. It allows you to write server-side code in Dart, automatically generate APIs, and hook up databases with minimal effort. This is a monorepo containing the core framework, CLI tools, modules, integrations, examples, and comprehensive tests.

## Pull Request Automation

When working with pull requests in this repository, **ALWAYS**:

1. **Use the PR description template**: Apply the exact template structure from `.github/pull_request_template.md`
2. **Reference issues**: Include "Fixes #XXXX." at the end of every PR description
3. **Follow conventional commits**: Ensure PR titles follow the format `<type>: <description>`
4. **Complete checklists**: Mark appropriate items in the pre-launch checklist
5. **Document breaking changes**: Always fill in the breaking changes section if applicable

### PR Template Enforcement

- **Never create** a PR description without using the standard template
- **Always preserve** the checklist structure and format
- **Ensure consistency** across all PRs for documentation quality
- **Reference the template file** directly when generating descriptions

### Documentation and Release Files

- **Never update** CHANGELOG files unless explicitly instructed by the user
- **Never update** README files unless explicitly instructed by the user
- CHANGELOG files are manually updated during release process
- Only make documentation changes when specifically requested

## Command Validation Status

**FULLY VALIDATED** (confirmed working in test environment):

- ✅ `docker compose config` - All Docker configurations are valid
- ✅ `docker compose up -d postgres redis` - Services start and are accessible
- ✅ `tests/docker/tests_integration/wait-for-it.sh` - Service readiness checker works
- ✅ Repository structure and file locations are accurate
- ✅ Script permissions and executability confirmed
- ✅ PostgreSQL and Redis connectivity tested (ports 8090, 8091)
- ✅ Example project Docker infrastructure verified

**REQUIRES DART/FLUTTER** (cannot validate without network access):

- ⚠️ `melos test_unit` - Requires Dart SDK
- ⚠️ `util/run_tests_analyze` - Requires Dart SDK
- ⚠️ `melos` commands - Requires Dart SDK and Melos package
- ⚠️ `dart pub get` operations - Require network access

**NOTE**: All timing estimates are based on CI pipeline analysis and repository examination.

## Prerequisites and Installation

**CRITICAL**: Install these exact versions to ensure compatibility:

### Required Tools
1. **Dart SDK**: `^3.11.3` (required)
2. **Flutter SDK**: `^3.41.2` (required for Flutter components)
3. **Git**: Any recent version
4. **Docker & Docker Compose**: Required for integration tests and local development
5. **Melos**: For monorepo management (`dart pub global activate melos`)
6. **Bash**: Required for scripts (use Git Bash on Windows)

### Installation Commands (Run these exactly)

```bash
# Install Flutter (choose one method)
# Method 1: Using FVM (Recommended if you have Dart already)
dart pub global activate fvm
fvm install 3.41.2
fvm use 3.41.2

# Method 2: Manual Flutter installation
cd /opt
sudo git clone https://github.com/flutter/flutter.git -b stable
export PATH="$PATH:/opt/flutter/bin"

# Method 3: Using snap (Ubuntu/Linux)
sudo snap install flutter --classic

# Install Melos for monorepo management
dart pub global activate melos

# Verify installations
dart --version     # Should be ^3.11.3
flutter --version  # Should be ^3.41.2
docker --version   # Should work
git --version      # Should work
```

**TROUBLESHOOTING**:

- If Dart SDK download fails during Flutter setup, try again or use a different network
- On CI/restricted networks, pre-install tools or use cached versions
- Ensure Docker daemon is running before running tests

## Repository Setup and Build Process

**NEVER CANCEL BUILDS OR LONG-RUNNING COMMANDS** - Builds may take 30-45 minutes, tests may take 15-60 minutes depending on scope.

### Initial Setup (Run from repository root)

```bash
# Alternative: melos bootstrap
dart pub global activate melos

# Install all dependencies - NEVER CANCEL.
melos bootstrap

# Activate CLI from source (required for development)
cd tools/serverpod_cli
dart pub get
dart pub global activate --source path .
cd ../..

# Set SERVERPOD_HOME for local templates
export SERVERPOD_HOME=$(pwd)
```

**TIMING**: `melos bootstrap` takes 10-15 minutes. Set timeout to 30+ minutes.

### Build Commands

```bash
# Generate all code (after model/endpoint changes)
util/generate_all

# Update pubspec files from templates (after template changes)
util/update_pubspecs

# Full repository dependency installation - NEVER CANCEL.
melos bootstrap
```

## Testing Infrastructure

**VERIFIED DOCKER TIMING**:

- **PostgreSQL container**: 5-10 seconds (first run with image download)
- **Redis container**: 2-5 seconds (first run with image download)
- **Subsequent starts**: 1-2 seconds per service

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serverpod/serverpod](https://github.com/serverpod/serverpod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
