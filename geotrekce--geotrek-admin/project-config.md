---
trigger: always_on
description: Geotrek-admin is a Django-based geospatial web application for managing trekking, outdoor tourism data and geographic information. It uses PostGIS for spatial data, Redis for caching, and provides both a web admin interface and comprehensive APIs.
---

# Geotrek-admin Development Instructions

Geotrek-admin is a Django-based geospatial web application for managing trekking, outdoor tourism data and geographic information. It uses PostGIS for spatial data, Redis for caching, and provides both a web admin interface and comprehensive APIs.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Network Limitations - CRITICAL
- **Docker build from source FAILS** due to firewall limitations preventing Python download from GitHub
- Timeout occurs when downloading `cpython-3.10.18+20250828-x86_64-unknown-linux-gnu-install_only_stripped.tar.gz`
- Build fails after exactly ~2 minutes with "operation timed out" error
- **WORKAROUND**: Use pre-built official images: `docker pull geotrekce/admin:latest`
- **Do not attempt** `docker compose build` - it will fail after ~2 minutes with network timeout
- **Alternative**: Use Ubuntu package installation method for full functionality

### Development Setup

#### Recommended: Development Target (Full Environment)
The `docker-compose.yml` at root is configured for development with `target: dev`. Use this for full development functionality:

```bash
# 1. Setup environment and permissions
cp .env.dist .env
mkdir -p var/{log,cache,media,static,tmp,conf}
chmod -R 777 var
echo "127.0.0.1 geotrek.local" | sudo tee -a /etc/hosts

# 2. Build and start the development environment
docker compose build  # NEVER CANCEL: Takes 5-10+ minutes, set timeout to 15+ minutes
docker compose up -d   # NEVER CANCEL: Takes 2-3 minutes, set timeout to 10+ minutes

# 3. Initialize the development environment
docker compose run --rm web update.sh     # NEVER CANCEL: Takes 2-5 minutes, set timeout to 15+ minutes
docker compose run --rm web load_data.sh  # NEVER CANCEL: Takes 5-15 minutes, set timeout to 30+ minutes
```

**TIMING**: Complete setup takes 15-30+ minutes. NEVER CANCEL during any step.

#### Alternative: Network-Limited Environment
If Docker build fails due to network restrictions, use the production image workaround:

```bash
# 1. Setup environment and permissions (same as above)
cp .env.dist .env
mkdir -p var/{log,cache,media,static,tmp,conf}
chmod -R 777 var
echo "127.0.0.1 geotrek.local" | sudo tee -a /etc/hosts

# 2. Pull pre-built production image and tag for local use
docker pull geotrekce/admin:latest
docker tag geotrekce/admin:latest geotrek:latest

# 3. Start supporting services only
docker compose up postgres redis convertit screamshotter -d
# Wait for postgres to be healthy (about 30 seconds)

# Note: Web container may not start properly with production image due to missing dev dependencies
```

**TIMING**: Service startup takes ~30-45 seconds. NEVER CANCEL during startup phase.

#### Alternative Installation Methods
For full functionality when Docker is not suitable:

1. **Ubuntu Package Installation** (Recommended for production-like setup):
   ```bash
   # Follow instructions from docs/installation-and-configuration/installation.rst
   bash -c "$(curl -fsSL https://raw.githubusercontent.com/GeotrekCE/Geotrek-admin/master/tools/install.sh)"
   ```

## Testing

### Django Tests
**CRITICAL TIMING**: Django tests take 15-30+ minutes. **NEVER CANCEL.** Set timeout to 45+ minutes minimum.

```bash
# Full test suite with coverage (LONG RUNNING - 20-30 minutes)
make coverage  # NEVER CANCEL: Takes 20-30 minutes. Set timeout to 45+ minutes

# Specific test environments  
make test      # NEVER CANCEL: Standard tests, ~15 minutes, set timeout to 30+ minutes
make test_nds  # NEVER CANCEL: Non-dynamic segmentation tests, ~15 minutes, set timeout to 30+ minutes  
make tests     # NEVER CANCEL: Both test and test_nds, ~30 minutes, set timeout to 60+ minutes
```

**Note**: Tests work with the development target (`target: dev` in docker-compose.yml). They may not work with the production image workaround due to missing development dependencies.

### Code Quality 
```bash
make quality   # ✅ Works with development target (dev), ❌ Fails with production image
make lint      # ✅ Works with development target (dev), ❌ Fails with production image  
make format    # ✅ Works with development target (dev), ❌ Fails with production image
```

**Note**: Quality commands work with the development target (`target: dev` in docker-compose.yml) which includes ruff and other dev tools. They fail with the production image workaround due to missing development dependencies.

### End-to-End Tests (Cypress)
**CRITICAL**: Requires complete setup with loaded data. Takes 10-15 minutes. **NEVER CANCEL.**

```bash
# Setup test data first (NEVER CANCEL each step)
make load_data              # NEVER CANCEL: ~5-10 minutes, set timeout to 20+ minutes
make load_test_integration  # NEVER CANCEL: ~2-5 minutes, set timeout to 15+ minutes

# Run Cypress tests (NEVER CANCEL)
cd cypress
npm ci                      # NEVER CANCEL: ~2-3 minutes first time, set timeout to 10+ minutes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeotrekCE/Geotrek-admin](https://github.com/GeotrekCE/Geotrek-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
