---
trigger: always_on
description: > **Note**: These instructions are for GitHub Copilot coding agent to help with code generation, issue resolution, and pull request improvements in this repository.
---

# ESP32 Project Template - GitHub Copilot Instructions

> **Note**: These instructions are for GitHub Copilot coding agent to help with code generation, issue resolution, and pull request improvements in this repository.

You are working on an ESP32 project template designed for rapid prototyping and production applications. This template provides a complete development environment with GitHub Codespaces, QEMU emulation, and example components for common IoT patterns.

## Project Context

### Template Purpose

This is a **template repository** designed to be:

- **Forked for new projects** - Starting point for ESP32 applications
- **Zero-setup development** - GitHub Codespaces with ESP-IDF pre-configured
- **Hardware optional** - QEMU emulation for testing without physical devices
- **Production-ready structure** - Component-based architecture following ESP-IDF best practices
- **Documentation included** - Sphinx-Needs requirements/design methodology

### Hardware Stack (When Using Real Hardware)

- **Microcontroller**: ESP32 (any variant supported by ESP-IDF)
- **Development Framework**: ESP-IDF v5.4.1
- **User adds their own**: Sensors, displays, or other peripherals

### Software Architecture

- **Component-based design**: Modular components in `main/components/`
- **Example components provided**:
  - `config_manager`: NVS configuration storage patterns
  - `web_server`: HTTP server with captive portal
  - `cert_handler`: HTTPS certificate handling (WIP)
  - `netif_uart_tunnel`: QEMU network bridge
- **Minimal main.c**: Template entry point users customize
- **Real-time OS**: FreeRTOS for task management
- **Memory optimized**: 4MB flash configuration

### Key Files and Components

- `main/main.c`: Application entry point (users customize this!)
- `main/components/config_manager/`: Example NVS configuration management
- `main/components/web_server/`: Example HTTP server with captive portal
- `main/components/cert_handler/`: HTTPS certificate handling (TODO: fix HTTPS)
- `main/components/netif_uart_tunnel/`: QEMU network bridge (required for emulation)

### Documentation Structure

- `docs/11_requirements/`: Sphinx-Needs requirements – **read these before implementing features**
- `docs/12_design/`: Component design specifications – **read the relevant spec before writing code**
- `docs/21_api/`: Auto-generated API docs per component
- `docs/90_guides/`: Developer guides (QEMU, flashing, switching dev modes, etc.)

> **Rule**: If a behaviour is specified in a requirements or design document, implement exactly that.
> Do **not** invent behaviour from general knowledge. When in doubt, check the relevant `.rst` files first and then consult with the user

## Development Guidelines

### Coding Standards

[ESP32 Coding Standards](./prompt-snippets/esp32-coding-standards.md)

### Build Instructions

[Detailed Build Instructions](./prompt-snippets/build-instructions.md)

### Development Workflow

[Development Process](./prompt-snippets/development.md)

### Commit Message Format

[Commit Message Guidelines](./prompt-snippets/commit-message.md)

## Project-Specific Instructions

### Memory Management

- **Always consider ESP32 memory constraints** when suggesting code changes
- **Use FreeRTOS-specific memory functions** (`heap_caps_malloc`, `heap_caps_free`)
- **Check available heap** before allocating large buffers
- **Prefer stack allocation** for small, temporary variables
- **Use IRAM_ATTR** sparingly and only when necessary

### API Documentation – Always Use Context7 First

Before writing or suggesting any FreeRTOS or ESP-IDF API calls, **always resolve the latest documentation via the context7 MCP server**:

- **FreeRTOS API**: use context7 library ID `/freertos/freertos`
- **ESP-IDF v5.4 API**: use context7 library ID `/websites/espressif_projects_esp-idf_en_v5_4_3_esp32c6`

This ensures correct function signatures, return types, and parameters for the exact versions used in this project (ESP-IDF v5.4.1, FreeRTOS as bundled).

### ESP-IDF Best Practices

- **Use component-based architecture** for new functionality
- **Include proper error handling** with ESP_ERROR_CHECK() and ESP_LOG functions
- **Follow ESP-IDF naming conventions** (esp_, ESP_, CONFIG_)
- **Use appropriate log levels** (ESP_LOGI, ESP_LOGW, ESP_LOGE, ESP_LOGD)
- **Handle WiFi events properly** using event handlers

### Hardware-Specific Considerations

- **Always document GPIO pin assignments** in the component or config header
- **Check ESP32 strapping pins** to avoid boot mode conflicts
- For all other hardware behaviour (timing, power, peripheral constraints): consult the relevant design spec in `docs/12_design/`

### Template-Specific Guidelines

- **Keep main.c minimal**: Application entry point should be simple and well-documented
- **Example components**: Maintain as reference implementations, not project code
- **Documentation**: Keep generic and applicable to any ESP32 project
- **QEMU support**: Ensure emulation works for testing without hardware
- **Codespaces first**: Prioritize GitHub Codespaces workflow over local setup

### Requirements Engineering


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enthali) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
