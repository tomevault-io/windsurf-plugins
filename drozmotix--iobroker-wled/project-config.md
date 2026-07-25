---
trigger: always_on
description: **Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions
---

# ioBroker Adapter Development with GitHub Copilot

**Version:** 0.5.6
**Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions

This file contains instructions and best practices for GitHub Copilot when working on ioBroker adapter development.

---

## 📑 Table of Contents

1. [Project Context](#project-context)
2. [Code Quality & Standards](#code-quality--standards)
   - [Code Style Guidelines](#code-style-guidelines)
   - [ESLint Configuration](#eslint-configuration)
3. [Testing](#testing)
   - [Unit Testing](#unit-testing)
   - [Integration Testing](#integration-testing)
   - [API Testing with Credentials](#api-testing-with-credentials)
4. [Development Best Practices](#development-best-practices)
   - [Dependency Management](#dependency-management)
   - [HTTP Client Libraries](#http-client-libraries)
   - [Error Handling](#error-handling)
5. [Admin UI Configuration](#admin-ui-configuration)
   - [JSON-Config Setup](#json-config-setup)
   - [Translation Management](#translation-management)
6. [Documentation](#documentation)
   - [README Updates](#readme-updates)
   - [Changelog Management](#changelog-management)
7. [CI/CD & GitHub Actions](#cicd--github-actions)
   - [Workflow Configuration](#workflow-configuration)
   - [Testing Integration](#testing-integration)

---

## Project Context

You are working on an ioBroker adapter. ioBroker is an integration platform for the Internet of Things, focused on building smart home and industrial IoT solutions. Adapters are plugins that connect ioBroker to external systems, devices, or services.

### WLED Adapter Specific Context

This adapter provides integration for WLED devices - ESP8266/ESP32-based LED controllers for NeoPixel (WS2812B, WS2811, SK6812, APA102) and SPI-based chipsets (WS2801).

**Key Characteristics:**
- **Device Discovery**: Uses Bonjour/mDNS service discovery to automatically detect WLED devices on the network
- **Communication**: HTTP/REST API and WebSocket connections for real-time updates
- **Device Control**: Controls LED effects, segments, colors (RGB/RGBW), brightness, and presets
- **Network Requirements**: Broadcast traffic must be routable for auto-discovery (VLAN separation can prevent this)
- **Fallback**: Manual device addition via IP address when auto-discovery fails

**Primary Dependencies:**
- `bonjour` (^3.5.0): mDNS/Bonjour service discovery for automatic device detection
- `axios` (^1.12.0): HTTP client for REST API communication with WLED devices
- `ws` (^8.7.0): WebSocket client for real-time device state updates
- `hex-rgb` (^4.3.0) & `rgb-hex` (^3.0.0): Color format conversion utilities for LED color handling

**Version Notes:**
- `bonjour`: v3.x provides stable mDNS discovery; no known breaking changes anticipated
- `axios`: v1.x is stable; maintains compatibility with WLED REST API patterns
- `ws`: v8.x is the current stable version; upgrade to v9+ should be carefully tested for WebSocket reconnection logic
- Color libraries: Current versions are stable and unlikely to change significantly

**Configuration Requirements:**
- Device list management (auto-discovered or manually added)
- Polling interval for device state updates (default: 30 seconds)
- Maximum retry attempts and backoff strategy for failed connections
- Individual device IP addresses and connection settings

**Special Considerations:**
- Handle offline/unavailable devices gracefully without crashing
- Reset device state (brightness to 0, power to false) when device disconnects or adapter starts
- Maintain connection state indicator for each device
- Support for multiple WLED devices simultaneously
- WebSocket reconnection logic for persistent real-time updates
- Proper cleanup of WebSocket connections in unload() (note: Bonjour browser cleanup not yet implemented)

---

## Code Quality & Standards

### Code Style Guidelines

- Follow JavaScript/TypeScript best practices
- Use async/await for asynchronous operations
- Implement proper resource cleanup in `unload()` method
- Use semantic versioning for adapter releases
- Include proper JSDoc comments for public methods

**Timer and Resource Cleanup Example:**
```javascript
private connectionTimer?: NodeJS.Timeout;

async onReady() {
  this.connectionTimer = setInterval(() => this.checkConnection(), 30000);
}

onUnload(callback) {
  try {
    if (this.connectionTimer) {
      clearInterval(this.connectionTimer);
      this.connectionTimer = undefined;
    }
    callback();
  } catch (e) {
    callback();
  }
}
```

### ESLint Configuration

**CRITICAL:** ESLint validation must run FIRST in your CI/CD pipeline, before any other tests. This "lint-first" approach catches code quality issues early.

#### Setup
```bash
npm install --save-dev eslint @iobroker/eslint-config
```

#### Configuration (.eslintrc.json)
```json
{
  "extends": "@iobroker/eslint-config",
  "rules": {
    // Add project-specific rule overrides here if needed
  }
}
```

#### Package.json Scripts
```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

#### Best Practices
1. ✅ Run ESLint before committing
2. ✅ Use `lint:fix` for auto-fixable issues
3. ✅ Don't disable rules without documentation
4. ✅ Lint all relevant files (main code, tests, build scripts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrozmotiX/ioBroker.wled](https://github.com/DrozmotiX/ioBroker.wled) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
