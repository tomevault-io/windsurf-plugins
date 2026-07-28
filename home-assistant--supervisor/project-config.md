---
trigger: always_on
description: This repository contains the Home Assistant Supervisor, a Python 3 based container
---

# GitHub Copilot & Claude Code Instructions

This repository contains the Home Assistant Supervisor, a Python 3 based container
orchestration and management system for Home Assistant.

## Pull Request Requirements

When opening a pull request in this repository, always use the description format
from `.github/PULL_REQUEST_TEMPLATE.md`.

## Supervisor Capabilities & Features

### Architecture Overview

Home Assistant Supervisor is a Python-based container orchestration system that
communicates with the Docker daemon to manage containerized components. It is tightly
integrated with the underlying Operating System and core Operating System components
through D-Bus.

**Managed Components:**
- **Home Assistant Core**: The main home automation application running in its own
  container (also provides the web interface)
- **Add-ons**: Third-party applications and services (each add-on runs in its own
  container)
- **Plugins**: Built-in system services like DNS, Audio, CLI, Multicast, and Observer
- **Host System Integration**: OS-level operations and hardware access via D-Bus
- **Container Networking**: Internal Docker network management and external
  connectivity
- **Storage & Backup**: Data persistence and backup management across all containers

**Key Dependencies:**
- **Docker Engine**: Required for all container operations
- **D-Bus**: System-level communication with the host OS
- **systemd**: Service management for host system operations
- **NetworkManager**: Network configuration and management

### Add-on System

**Add-on Architecture**: Add-ons are containerized applications available through
add-on stores. Each store contains multiple add-ons, and each add-on includes metadata
that tells Supervisor the version, startup configuration (permissions), and available
user configurable options. Add-on metadata typically references a container image that
Supervisor fetches during installation. If not, the Supervisor builds the container
image from a Dockerfile.

**Built-in Stores**: Supervisor comes with several pre-configured stores:
- **Core Add-ons**: Official add-ons maintained by the Home Assistant team
- **Community Add-ons**: Popular third-party add-ons repository
- **ESPHome**: Add-ons for ESPHome ecosystem integration
- **Music Assistant**: Audio and music-related add-ons
- **Local Development**: Local folder for testing custom add-ons during development

**Store Management**: Stores are Git-based repositories that are periodically updated.
When updates are available, users receive notifications.

**Add-on Lifecycle**:
- **Installation**: Supervisor fetches or builds container images based on add-on
  metadata
- **Configuration**: Schema-validated options with integrated UI management
- **Runtime**: Full container lifecycle management, health monitoring
- **Updates**: Automatic or manual version management

### Update System

**Core Components**: Supervisor, Home Assistant Core, HAOS, and built-in plugins
receive version information from a central JSON file fetched from
`https://version.home-assistant.io/{channel}.json`. The `Updater` class handles
fetching this data, validating signatures, and updating internal version tracking.

**Update Channels**: Three channels (`stable`/`beta`/`dev`) determine which version
JSON file is fetched, allowing users to opt into different release streams.

**Add-on Updates**: Add-on version information comes from store repository updates, not
the central JSON file. When repositories are refreshed via the store system, add-ons
compare their local versions against repository versions to determine update
availability.

### Backup & Recovery System

**Backup Capabilities**:
- **Full Backups**: Complete system state capture including all add-ons,
  configuration, and data
- **Partial Backups**: Selective backup of specific components (Home Assistant,
  add-ons, folders)
- **Encrypted Backups**: Optional backup encryption with user-provided passwords
- **Multiple Storage Locations**: Local storage and remote backup destinations

**Recovery Features**:
- **One-click Restore**: Simple restoration from backup files
- **Selective Restore**: Choose specific components to restore
- **Automatic Recovery**: Self-healing for common system issues

---

## Supervisor Development

### Python Requirements

  - **Compatibility**: Python 3.14+
  - **Language Features**: Use modern Python features:
  - Type hints with `typing` module
  - f-strings (preferred over `%` or `.format()`)
  - Dataclasses and enum classes
  - Async/await patterns
  - Pattern matching where appropriate
  - Parenthesis-free `except` clauses with comma-separated exceptions
    (e.g., `except KeyError, TypeError:`) — available since Python 3.14

### Code Quality Standards

- **Formatting**: Ruff
- **Linting**: PyLint and Ruff  
- **Type Checking**: MyPy
- **Testing**: pytest with asyncio support
- **Language**: American English for all code, comments, and documentation

### Code Organization

**Core Structure**:
```
supervisor/
├── __init__.py           # Package initialization
├── const.py             # Constants and enums
├── coresys.py           # Core system management
├── bootstrap.py         # System initialization
├── exceptions.py        # Custom exception classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [home-assistant/supervisor](https://github.com/home-assistant/supervisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
