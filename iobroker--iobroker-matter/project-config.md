---
trigger: always_on
description: This is an ioBroker adapter for integrating Matter devices into the ioBroker home automation platform. The adapter supports both bridging ioBroker devices to Matter networks and controlling external Matter devices from ioBroker.
---

# GitHub Copilot Instructions for ioBroker.matter

## Project Overview

This is an ioBroker adapter for integrating Matter devices into the ioBroker home automation platform. The adapter supports both bridging ioBroker devices to Matter networks and controlling external Matter devices from ioBroker.

### Key Features
- Bridge ioBroker devices to Matter controllers (like Apple HomeKit, Google Home, Amazon Alexa)
- Connect and control external Matter devices
- Support for various device types (lights, sensors, switches, thermostats, etc.)
- Web-based admin interface for configuration
- Device auto-discovery and pairing

## Technology Stack

### Backend (src/)
- **Language**: TypeScript
- **Runtime**: Node.js 20+
- **Framework**: ioBroker Adapter Core (@iobroker/adapter-core)
- **Matter Library**: @matter/main, @project-chip/matter.js
- **Device Detection**: @iobroker/type-detector
- **Testing**: Mocha

### Frontend (src-admin/)
- **Language**: TypeScript/TSX
- **Framework**: React (class components)
- **UI Library**: Material-UI (@mui/material)
- **Build Tool**: Vite
- **State Management**: Component state (no Redux/Context)
- **Config**: @iobroker/json-config for dynamic forms

### Build & Development Tools
- **TypeScript Compiler**: tsc
- **Linting**: ESLint with @iobroker/eslint-config
- **Formatting**: Prettier
- **Testing**: Mocha with @iobroker/testing
- **CI/CD**: GitHub Actions

## Project Structure

```
├── src/                          # Backend TypeScript source
│   ├── main.ts                   # Main adapter entry point
│   ├── lib/                      # Shared libraries and utilities
│   │   ├── DeviceManagement.ts   # Device management logic
│   │   ├── devices/              # Device-specific implementations
│   │   └── i18n/                 # Internationalization files
│   └── matter/                   # Matter protocol implementations
│       ├── ControllerNode.ts     # Matter controller logic
│       ├── DeviceNode.ts         # Matter device implementations  
│       ├── BridgedDevicesNode.ts # Bridging logic
│       ├── GeneralMatterNode.ts  # Base Matter node functionality
│       ├── BaseServerNode.ts     # Base server node implementation
│       ├── behaviors/            # Custom Matter.js behaviors for ioBroker integration
│       │   ├── IoBrokerContext.ts           # Context behavior for sharing adapter/device state
│       │   ├── IoBrokerEvents.ts            # Event system for ioBroker-Matter communication
│       │   ├── EventedOnOffLightOnOffServer.ts # Event-driven OnOff server behavior
│       │   ├── EventedLightingLevelControlServer.ts # Event-driven dimming behavior
│       │   ├── ThermostatServer.ts          # Custom thermostat behavior
│       │   └── PowerSourceServer.ts         # Battery and power management behavior
│       ├── to-iobroker/          # Matter device → ioBroker state mapping (Controller mode)
│       │   ├── ioBrokerFactory.ts           # Factory for creating ioBroker devices from Matter
│       │   ├── GenericDeviceToIoBroker.ts   # Base class for Matter→ioBroker mapping
│       │   ├── OnOffLightToIoBroker.ts      # Maps Matter lights to ioBroker light states
│       │   ├── DimmableToIoBroker.ts        # Maps Matter dimmable devices
│       │   ├── TemperatureSensorToIoBroker.ts # Maps Matter temperature sensors
│       │   ├── ThermostatToIoBroker.ts      # Maps Matter thermostats
│       │   └── [Other device mappers]       # Device-specific mapping implementations
│       └── to-matter/            # ioBroker device → Matter device mapping (Bridge mode)
│           ├── matterFactory.ts             # Factory for creating Matter devices from ioBroker
│           ├── GenericDeviceToMatter.ts     # Base class for ioBroker→Matter mapping
│           ├── LightToMatter.ts             # Maps ioBroker lights to Matter endpoints
│           ├── DimmerToMatter.ts            # Maps ioBroker dimmers to Matter dimmable lights
│           ├── TemperatureToMatter.ts       # Maps ioBroker temperature sensors
│           ├── ThermostatToMatter.ts        # Maps ioBroker thermostats to Matter thermostats
│           └── [Other device mappers]       # Device-specific mapping implementations
├── src-admin/                    # Frontend React admin interface
│   ├── src/
│   │   ├── App.tsx               # Main application component
│   │   ├── Tabs/                 # Tab components (Options, Bridges, Devices)
│   │   └── components/           # Reusable UI components
│   └── package.json              # Frontend dependencies
├── build/                        # Compiled backend code
├── admin/                        # Compiled frontend assets
├── test/                         # Test files
├── tasks.js                      # Custom build scripts
└── io-package.json               # ioBroker adapter metadata
```

## Coding Conventions

### TypeScript/JavaScript
- Use TypeScript strict mode
- Prefer explicit type annotations for public APIs
- Use async/await over Promises when possible
- Follow ESLint rules from @iobroker/eslint-config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ioBroker/ioBroker.matter](https://github.com/ioBroker/ioBroker.matter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
