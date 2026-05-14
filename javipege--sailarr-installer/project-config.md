---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sailarr Installer is an automated Docker-based media streaming stack that leverages Real-Debrid and the *Arr ecosystem to create an "infinite" media library. This is a fully automated installer with a modular template system that deploys a microservices architecture using Docker Compose to orchestrate services including Plex, Overseerr, Radarr, Sonarr, Prowlarr, Zilean, Zurg, Decypharr, Recyclarr, Autoscan, Tautulli, Homarr, Pinchflat, PlexTraktSync, and Watchtower.

The installer is designed to be run once (`./setup.sh`) and handles everything from user input collection to full stack deployment and service configuration with zero manual steps required.

## Essential Commands

### Initial Setup (Run Once)
```bash
chmod +x setup.sh
./setup.sh
sudo reboot  # Required after setup
```

### Stack Management
```bash
# Navigate to docker directory
cd /YOUR_INSTALL_DIR/docker

# Start the entire stack (using helper scripts - recommended)
./up.sh
./down.sh
./restart.sh

# Or using docker compose directly with required env files
docker compose --env-file .env.defaults --env-file .env.local up -d
docker compose --env-file .env.defaults --env-file .env.local down
docker compose --env-file .env.defaults --env-file .env.local restart [service_name]

# For Traefik profile
docker compose --env-file .env.defaults --env-file .env.local --profile traefik up -d

# Monitor logs
docker compose logs -f [service_name]

# Update quality profiles
/YOUR_INSTALL_DIR/scripts/recyclarr-sync.sh
```

### Debugging and Monitoring
```bash
# Check service health
docker ps -a

# View specific service logs
docker logs radarr
docker logs sonarr
docker logs zurg

# Monitor container resources
docker stats

# Check health check logs
tail -f /YOUR_INSTALL_DIR/logs/plex-mount-healthcheck.log
tail -f /YOUR_INSTALL_DIR/logs/arrs-mount-healthcheck.log

# Verify cron jobs
crontab -l | grep healthcheck

# Manual health check execution
/YOUR_INSTALL_DIR/scripts/health/plex-mount-healthcheck.sh
/YOUR_INSTALL_DIR/scripts/health/arrs-mount-healthcheck.sh
```

## Architecture & Key Concepts

### Data Flow Pattern
The system uses a **symlink-based architecture** optimized for hardlinking:
1. **Request**: Overseerr → Radarr/Sonarr → Prowlarr → Zilean/Torrentio/Public Indexers
2. **Download**: Decypharr → Real-Debrid → Zurg → Rclone Mount
3. **Media**: Symlinks → Media folders → Plex → Autoscan refresh → PlexTraktSync tracking

### Services List

**Core Media Stack:**
- **Plex** - Media server (host network mode)
- **Overseerr** - Request management interface (port 5055)
- **Radarr** - Movie management (port 7878)
- **Sonarr** - TV show management (port 8989)
- **Prowlarr** - Indexer manager (port 9696)

**Download & Storage:**
- **Zurg** - Real-Debrid WebDAV interface (port 9999)
- **Rclone** - Mount Real-Debrid storage
- **Decypharr** - Download client with Debrid integration (port 8282)

**Indexers:**
- **Zilean** - DMM torrent indexer (port 8181)
- **Torrentio** - Stremio indexer integration
- **Public Indexers** - 1337x, TPB, YTS, EZTV

**Automation & Monitoring:**
- **Recyclarr** - Automated quality profiles via TRaSH Guides
- **Autoscan** - Plex library auto-update (port 3030)
- **Tautulli** - Plex statistics and monitoring (port 8282)
- **PlexTraktSync** - Sync Plex watch history to Trakt
- **Watchtower** - Automatic container updates
- **Homarr** - Dashboard for all services (port 7575)
- **Pinchflat** - YouTube download manager (port 8945)

**Optional:**
- **Traefik** - Reverse proxy with HTTPS (ports 80, 443) - requires network configuration

### Directory Structure
```
${ROOT_DIR}/
├── config/              # Container configurations (created by setup.sh)
│   ├── plex-config/
│   ├── radarr-config/
│   ├── sonarr-config/
│   ├── prowlarr-config/
│   ├── overseerr-config/
│   ├── zilean-config/
│   ├── zurg-config/
│   ├── autoscan-config/
│   ├── decypharr-config/
│   ├── tautulli-config/
│   ├── homarr-config/
│   ├── pinchflat-config/
│   ├── plextraktsync-config/
│   └── traefik-config/  # Only if Traefik enabled
├── data/
│   ├── media/
│   │   ├── movies/      # Radarr movies
│   │   ├── tv/          # Sonarr TV shows
│   │   ├── radarr/      # Radarr symlinks for downloads
│   │   └── sonarr/      # Sonarr symlinks for downloads
│   └── realdebrid-zurg/ # Rclone mount point
└── logs/                # Health check logs
```

### Repository Structure
```
/repository-root/
├── setup.sh             # Main installation script - entry point
├── .env.install         # User configuration (created by setup.sh)
├── README.md            # User documentation
├── INSTALLATION.md      # Detailed installation guide
├── LICENSE              # MIT License
├── CLAUDE.md            # This file - Claude Code guidance
│
├── setup/               # Setup scripts and libraries
│   ├── lib/            # Modular function libraries (sourced by setup.sh)
│   │   ├── setup-common.sh    # Logging, validation, wait functions
│   │   ├── setup-users.sh     # User/group management
│   │   ├── setup-api.sh       # Generic API functions
│   │   └── setup-services.sh  # High-level service config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JaviPege/sailarr-installer](https://github.com/JaviPege/sailarr-installer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
