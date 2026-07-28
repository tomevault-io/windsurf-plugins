---
trigger: always_on
description: Instructions for AI coding agents working with this Call of Duty 2 dedicated server repository.
---

# AGENTS.md

Instructions for AI coding agents working with this Call of Duty 2 dedicated server repository.

## Project Overview

Containerized COD2 multiplayer dedicated server with libcod support using Docker multi-stage builds (Debian build → Alpine runtime).

**Architecture:**

- **17 server variants**: versions 1.0/1.2/1.3 with various patches and libcod libraries (binaries in [bin/](bin/))
- **Multi-variant build**: All variants built via matrix strategy from [.github/config/variants.json](.github/config/variants.json)
- **Default variant**: `1_3_nodelay_va_loc-ibuddieat` (v1.3 + nodelay + VA security + no localization spam + ibuddieat libcod)
- **Build args control variant**: `COD2_VERSION` (e.g., `1_3`), `COD2_LNXDED_TYPE` (e.g., `_nodelay_va_loc`), `LIBCOD_TYPE` (e.g., `ibuddieat`)

**Key Binaries & Suffixes:**

- `_nodelay`: Reduces master server reconnect (30min → 5sec)
- `_va`: VA security patch for buffer overflow (recommended)
- `_loc`: Suppresses non-localized string spam
- `_cracked`: Disables master server + nodelay
- See [bin/readme.md](bin/readme.md) for details

## Quick Reference

### Development Scripts

| Command | Purpose |
|---------|---------|
| `./scripts/dev-build.sh` | Build local image |
| `./scripts/dev-up.sh` | Build and start server (detached) |
| `./scripts/dev-logs.sh` | View real-time logs |
| `./scripts/dev-attach.sh` | Attach to console (detach: CTRL+P, CTRL+Q) |
| `./scripts/dev-exec.sh` | Open shell in container |
| `./scripts/dev-down.sh` | Stop and cleanup |

### Testing & Linting

**Important:** Always run tests and linting locally before committing.

```bash
# Lint
docker run --rm -i hadolint/hadolint < Dockerfile
docker run --rm -v "$PWD:/mnt" koalaman/shellcheck:stable scripts/*.sh

# Test
./container-structure-test-linux-amd64 test --image bgauduch/cod2server:dev --config tests/container-structure-test.yaml
./tests/test-container-health.sh bgauduch/cod2server:dev
```

## Build & Release System

### Binary Naming Convention

Format: `cod2_lnxded_X_Y[suffix]` (located in [bin/](bin/))

- `X_Y`: Version (`1_0`, `1_2`, `1_3`)
- `[suffix]`: Optional modifiers (`_nodelay`, `_va`, `_loc`, `_cracked`, `a`, `c`)

### Docker Image Tags

**Immutable:** `bgauduch/cod2server:6.0.0-1_3_nodelay_va_loc-ibuddieat` (semver + variant + libcod)
**Mutable:** `latest`, `6`, `6.0`, `1_3_nodelay_va_loc-ibuddieat`, `1_3_nodelay_va_loc-voron`

**Format:** `COD2_VERSION + COD2_LNXDED_TYPE + "-" + LIBCOD_TYPE`

### CI/CD Pipeline

[.github/workflows/build-test-push.yml](.github/workflows/build-test-push.yml):

1. Load config from [.github/config/variants.json](.github/config/variants.json)
1. Build all 17 variants (matrix strategy)
1. Run container structure tests + Trivy security scan
1. Save artifacts (zstd compression)
1. Push on release with semantic version tags

**Critical:** All variants built on every PR to ensure universal compatibility.

### Workflows

| Workflow | Trigger | Actions |
|----------|---------|---------|
| [Lint](.github/workflows/lint.yml) | PR/push | Hadolint (Dockerfile) + ShellCheck (scripts) |
| [Build/Test/Push](.github/workflows/build-test-push.yml) | PR/push/release | Build all 17 variants, test, scan, push |
| [Release Please](.github/workflows/release-please.yml) | Push to main | Create release PR, update CHANGELOG, trigger builds |

Path-filtered workflows above have companion `skip-*.yml` workflows that report the same *required* status checks as no-op successes for PRs that don't touch the relevant paths. This prevents required-but-skipped checks from getting stuck in a permanent "Expected" state. Keep each `skip-*.yml` `paths-ignore` list and job `name:` values in sync with the real workflow it mirrors.

### Conventional Commits

| Type | Version Bump | Example |
|------|--------------|---------|
| `feat:` | Minor (x.Y.0) | `feat: add new map rotation` |
| `fix:` | Patch (x.x.Y) | `fix: resolve connection timeout` |
| `feat!:` or `BREAKING CHANGE:` | Major (X.0.0) | `feat!: change config format` |
| `docs:`, `refactor:`, `perf:`, `chore:` | Patch | Categorized in changelog |

See [.github/RELEASE_PROCESS.md](.github/RELEASE_PROCESS.md) for details.

## Project Structure

```text
bin/                      # COD2 server binaries
lib/pb/                   # PunkBuster files (v1.760)
cod2server/main/          # Volume mount for game files (.iwd)
Dockerfile                # Multi-stage build (voron/ibuddieat builders)
docker-compose.yaml       # Production config
docker-compose.dev.yaml   # Development override
scripts/                  # Dev helper scripts
tests/                    # Container validation
.github/
  ├── workflows/          # CI/CD pipelines
  ├── config/variants.json    # Variant definitions (17 combinations)
  └── actions/setup-config/   # Reusable config loader
```

## Configuration Reference

### Dockerfile Build Arguments

All build arguments have sensible defaults for quick local development:

- `COD2_VERSION` - Server version (`1_0`, `1_2`, `1_3`) [default: `1_3`]
- `COD2_LNXDED_TYPE` - Binary variant suffix (`_nodelay_va_loc`, `a`, `c`, etc.) [default: `_nodelay_va_loc`]
- `LIBCOD_TYPE` - LibCOD library variant (`voron`, `ibuddieat`) [default: `ibuddieat`]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bgauduch/call-of-duty-2-docker-server](https://github.com/bgauduch/call-of-duty-2-docker-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
