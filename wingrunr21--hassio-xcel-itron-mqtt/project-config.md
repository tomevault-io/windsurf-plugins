---
trigger: always_on
description: This file provides structured guidance for AI agents working with this codebase.
---

# AGENTS.md

This file provides structured guidance for AI agents working with this codebase.

## Project Context

**Purpose**: Home Assistant addon that bridges Xcel Energy iTron Riva Gen 5 smart meters to MQTT

**Technology Stack**:
- Home Assistant addon framework
- Python (via [zaknye/xcel_itron2mqtt](https://github.com/zaknye/xcel_itron2mqtt))
- Docker multi-stage builds
- S6 overlay for service management
- IEEE 2030.5 protocol for meter communication
- MQTT for Home Assistant integration

**Repository Structure**:
```
xcel-itron-mqtt/         # Stable addon version
xcel-itron-mqtt-edge/    # Edge addon version (pre-release)
scripts/                 # Build and maintenance utilities
.devcontainer/           # Development container configuration
```

## Agent Rules

### Code Modification Principles

1. **Home Assistant Conventions**: Always follow Home Assistant addon patterns and standards
2. **Shell Script Standards**:
   - Use `#!/command/with-contenv bashio` shebang for integration with Home Assistant
   - Leverage bashio library for all logging and configuration access
   - Follow shellcheck standards (pre-configured in devcontainer)
3. **Certificate Security**: EC P-256 curve certificates are required by IEEE 2030.5 - do not change certificate generation algorithm. Generate certs in the `init-xcel-itron-mqtt` service, not upstream's `scripts/generate_keys.sh` — the addon supports configurable `cert_file`/`key_file` names and writes the computed LFDI back to the addon config via `bashio::addon.option`, neither of which the upstream script does
4. **Version Synchronization**: When updating base images or dependencies, update both stable and edge versions unless explicitly instructed otherwise
5. **Upstream Dependency Management**:
   - Python code comes from upstream GitHub via commit SHA
   - Update `XCEL_ITRON2MQTT_SHA` in Dockerfile to change versions
   - Always remove upstream `run.sh` and use S6 overlay service scripts instead
   - Install dependencies with `uv sync --frozen --no-dev --no-install-project` from the upstream `pyproject.toml` + `uv.lock`. The Dockerfile copies a pinned `uv` binary via `COPY --from=ghcr.io/astral-sh/uv:<version>` and must copy `pyproject.toml` and `uv.lock` out of the upstream archive into `/opt/xcel_itron2mqtt`. Put the resulting `.venv/bin` on `PATH` so the S6 service's `python3 main.py` runs from the uv-managed virtualenv.

### Testing Requirements

- No automated test suite exists - all changes require manual testing
- Test using Home Assistant development container for integration validation
- Ideally test with actual iTron meter hardware or mock server
- Verify MQTT integration after any protocol or communication changes

## Architecture Reference

### Addon Components

**Configuration Files**:
- `config.yaml` - Addon metadata, user configuration schema, and options
- `build.yaml` - Docker build settings and base image specifications
- `Dockerfile` - Multi-stage build downloading upstream Python code
- `rootfs/` - Files deployed to addon container filesystem

**Service Structure** (S6 overlay in `rootfs/etc/s6-overlay/s6-rc.d/`):
- **init-xcel-itron-mqtt**: One-time initialization service
  - Generates SSL certificates using OpenSSL
  - Calculates LFDI (Local Device Functional Identifier) from certificate
  - Runs before main service starts
- **xcel-itron-mqtt**: Main application service
  - Runs Python bridge connecting meter to MQTT
  - Depends on init service completion

### Data Flow

1. Addon connects to iTron smart meter over local network (IEEE 2030.5 protocol)
2. Meter data collected and formatted by Python library
3. Data published to Home Assistant's MQTT broker
4. Home Assistant auto-discovers devices via MQTT discovery protocol

### Certificate Management

- **Algorithm**: EC P-256 curve (required by IEEE 2030.5 standard)
- **Storage**: `/config/certs` directory (persisted in Home Assistant addon_configs)
- **LFDI Calculation**: First 40 characters of certificate SHA-256 fingerprint
- **Auto-generation**: Init service creates certificates on first run if missing

### Configuration Schema

**User-configurable options** (`config.yaml`):
- `meter_ip`: Required - iTron meter IP address
- `meter_port`: Default 8081 - iTron meter communication port
- `cert_dir`: Certificate storage location (default `/config/certs`)
- `lfdi`: Auto-calculated from certificates (read-only)

**Build configuration** (`build.yaml`):
- Base image: Home Assistant base-python image
- Architectures: aarch64, amd64

## Common Development Tasks

### Building Addons Locally

**Using utility scripts** (recommended):
```bash
# Build both stable and edge versions
./scripts/build-local.sh both

# Build only stable version
./scripts/build-local.sh stable

# Build edge version without cache
./scripts/build-local.sh edge --no-cache
```

**Manual Docker builds**:
```bash
# Stable addon (check build.yaml for current base image version)
cd xcel-itron-mqtt
docker build --build-arg BUILD_FROM="ghcr.io/hassio-addons/base-python:18.0.0" \
  -t local/hassio-xcel-itron-mqtt .

# Edge addon
cd xcel-itron-mqtt-edge
docker build --build-arg BUILD_FROM="ghcr.io/hassio-addons/base-python:18.0.0" \
  -t local/hassio-xcel-itron-mqtt-edge .
```

### Updating Base Images


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wingrunr21/hassio-xcel-itron-mqtt](https://github.com/wingrunr21/hassio-xcel-itron-mqtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
