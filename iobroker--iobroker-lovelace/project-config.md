---
trigger: always_on
description: **Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions
---

# ioBroker.lovelace Copilot Instructions

**Version:** 0.4.0  
**Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions

This file contains instructions and best practices for GitHub Copilot when working on the ioBroker.lovelace adapter development, integrating both adapter-specific context and community-standard ioBroker development patterns.

## ioBroker Context

You are working on an ioBroker adapter. ioBroker is an integration platform for the Internet of Things, focused on building smart home and industrial IoT solutions. Adapters are plugins that connect ioBroker to external systems, devices, or services.

**This specific adapter** provides visualization capabilities using the Home Assistant Lovelace UI, bridging ioBroker smart home objects with the modern, card-based Lovelace interface originally developed for Home Assistant.

## Project Overview

This is the **ioBroker.lovelace** adapter - an ioBroker adapter that provides visualization capabilities using the Home Assistant Lovelace UI. It bridges ioBroker smart home objects with the modern, card-based Lovelace interface originally developed for Home Assistant.

### Key Purpose
- Provides a modern web-based visualization interface for ioBroker installations
- Allows users to create dashboards using the flexible Lovelace card system
- Automatically detects ioBroker objects and converts them to Home Assistant-compatible entities
- Supports custom cards, themes, and advanced visualization features

## Architecture & Key Concepts

### ioBroker Adapter Pattern
- **main.js**: Entry point following ioBroker adapter conventions
- **io-package.json**: Adapter metadata and configuration schema
- **package.json**: Standard Node.js package configuration

### Core Components

#### 1. Entity System (`lib/entities/`, `lib/converters/`)
- **Entities**: ioBroker objects converted to Home Assistant format
- **Converters**: Transform ioBroker object types to HA entity types:
  - `switch.js` - Switch entities
  - `light.js` - Light entities with dimming, color, etc.
  - `sensor.js` - Sensor readings
  - `binary_sensor.js` - On/off sensors
  - `climate.js` - Thermostats and HVAC
  - `cover.js` - Blinds, shutters, garage doors
  - `media_player.js` - Media players
  - `weather.js` - Weather stations
  - `camera.js` - Camera/image entities
  - `alarm_control_panel.js` - Security systems

#### 2. Server System (`lib/server.js`)
- Express.js-based web server
- WebSocket communication for real-time updates
- Authentication and session management
- API endpoints for frontend communication

#### 3. Modules (`lib/modules/`)
- **autoEntities.js**: Automatic entity detection from ioBroker objects
- **browser_mod.js**: Browser control functionality (popup, notification, etc.)
- **dashboard.js**: Dashboard management
- **entityRegistry.js**: Entity registration and ID management
- **history.js**: Historical data handling
- **statisticsRecorder.js**: Long-term statistics
- **persistentNotifications.js**: System notifications
- **todo.js**: Task/shopping list functionality

#### 4. Frontend (`hass_frontend/`)
- Modified Home Assistant frontend
- Custom cards and UI components
- Static assets and resources

## File Structure

```
/
├── main.js                    # Adapter entry point
├── io-package.json           # Adapter configuration
├── package.json              # Node.js dependencies
├── lib/                      # Core adapter logic
│   ├── server.js            # Main server implementation
│   ├── entities/            # Entity management
│   ├── converters/          # Object-to-entity converters
│   ├── modules/             # Feature modules
│   └── tools.js             # Utility functions
├── admin/                   # Admin UI files
│   ├── words.js            # Translations
│   └── i18n/               # Localization files
├── hass_frontend/          # Modified Home Assistant frontend
├── docs/                   # Documentation (multi-language)
├── test/                   # Test files
└── gulpfile.js            # Build tasks
```

## Development Patterns & Conventions

### Entity Conversion Pattern
```javascript
// Standard pattern for converter functions
function processEntityType(id, obj, entity, objects, customsData) {
    // 1. Validate object structure
    // 2. Extract states and attributes
    // 3. Map to Home Assistant entity format
    // 4. Apply custom configurations
    // 5. Return formatted entity
}
```

### Module Pattern
```javascript
// Standard module pattern used throughout
class ModuleName {
    constructor(adapter) {
        this.adapter = adapter;
        this.entities = new Map();
    }
    
    async init() {
        // Module initialization
    }
    
    destroy() {
        // Cleanup resources
    }
    
    // Optional functions - will be automatically called by server.js if present
    async processMessage(ws, message) {
        // Handle WebSocket messages from frontend
        // Return true if message was handled
    }
    
    onStateChange(id, state, wss) {
        // Handle ioBroker state changes
        // Called when subscribed states change
    }
    
    onObjectChange(id, obj) {
        // Handle ioBroker object changes  
        // Called when subscribed objects change
    }
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ioBroker/ioBroker.lovelace](https://github.com/ioBroker/ioBroker.lovelace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
