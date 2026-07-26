---
trigger: always_on
description: This document provides essential information for AI coding agents working on this project.
---

# Agent Documentation for ambientweather2mqtt

This document provides essential information for AI coding agents working on this project.

## Project Overview

**ambientweather2mqtt** is a Node.js service that listens for local data from Ambient Weather stations (such as the WS-2902C) and converts it to MQTT events for Home Assistant integration. The sensor data is published with auto-discovery so sensors appear automatically in Home Assistant.

## Tech Stack

- **Language**: TypeScript (compiled to JavaScript)
- **Runtime**: Node.js 22+ (Alpine Linux for Docker)
- **Package Manager**: npm
- **Web Framework**: Express.js
- **MQTT Client**: async-mqtt
- **Testing**: Mocha + Chai
- **Build System**: TypeScript Compiler (tsc)
- **Code Quality**: ESLint (flat config) + Prettier + Markdownlint
- **Deployment**: Docker, Home Assistant Add-on

## Project Structure

```text
ambientweather2mqtt/
├── src/                          # TypeScript source code
│   ├── main.ts                   # Application entry point
│   ├── env.ts                    # Environment variable validation (Zod schema)
│   ├── webServer.ts              # Express server for receiving weather data
│   ├── mqttManager.ts            # MQTT client and publishing logic
│   ├── entityManager.ts          # Home Assistant entity management
│   ├── sensor.ts                 # Sensor entity class
│   ├── binarySensor.ts           # Binary sensor entity class
│   ├── calculations.ts           # Calculated sensor values (dewpoint, feels like, etc.)
│   ├── entityNames.ts            # Enum of all entity names
│   ├── deviceClass.ts            # Home Assistant device classes
│   ├── sensorUnit.ts             # Sensor units of measurement
│   ├── controllers/              # Express route controllers
│   └── types/                    # TypeScript type definitions
│       └── environment.d.ts      # Process.env type definitions
├── test/                         # Test files
│   ├── calculatedSensors.spec.ts # Tests for calculated sensors
│   └── test.env                  # Test environment variables
├── dist/                         # Compiled JavaScript output (gitignored)
├── hassio_aw2m/                  # Home Assistant add-on files
│   ├── Dockerfile                # Home Assistant-specific Docker image
│   ├── config.yaml               # Add-on configuration
│   ├── startup.sh                # Add-on startup script
│   └── CHANGELOG.md              # Add-on changelog
├── sampleConfiguration/          # Example configuration files
│   ├── .env                      # Sample environment variables
│   └── docker-compose.yml        # Sample Docker Compose setup
├── docs/                         # Documentation assets
├── Dockerfile                    # Main Docker image (Node 22 Alpine)
├── package.json                  # npm dependencies and scripts
├── tsconfig.json                 # TypeScript configuration
├── eslint.config.mjs             # ESLint flat configuration
├── .nvmrc                        # Node version specification (22)
├── .prettierrc                   # Prettier configuration
├── .markdownlint.json            # Markdownlint configuration
├── .mocharc.jsonc                # Mocha test configuration
├── README.md                     # User documentation
├── DEVELOPMENT.md                # Developer documentation
└── CHANGELOG.md                  # Project changelog
```

## Environment Configuration

Required environment variables (defined in `src/env.ts`):

- `MQTT_SERVER`: MQTT server URL (e.g., `http://192.168.1.1:1883`)
- `STATION_MAC_ADDRESS`: MAC address of the Ambient Weather station
- `TZ`: Timezone (e.g., `America/Los_Angeles`)
- `PORT`: Port for the web server (default: `8132`)

Optional environment variables:

- `MQTT_USERNAME`: MQTT authentication username
- `MQTT_PASSWORD`: MQTT authentication password
- `MQTT_REJECT_UNAUTHORIZED`: TLS certificate validation (`true`|`false`, default: `false`)
- `LOG_LEVEL`: Logging level (`error`|`warn`|`info`|`http`|`debug`|`trace`, default: `info`)
- `LOCALE`: Date/time formatting locale (default: `en-US`)
- `TOPIC_ROOT`: MQTT topic root (optional)
- `PUBLISH_NAME`: Device name for MQTT (default: `ambientWeather2mqtt`)
- `RETAIN_SENSOR_VALUES`: Retain MQTT messages (`true`|`false`, default: `false`)
- `NODE_ENV`: Node environment (default: `production`)
- `VERSION`: Application version (default: `dev`)

Environment variables are validated using Zod schemas in `src/env.ts`.

## Development Workflow

### Initial Setup

```bash
npm install      # Install dependencies and build (runs prepare script)
```

### Building

```bash
npm run build    # Compile TypeScript to dist/
```

TypeScript is compiled to ES modules in the `dist/` directory using the configuration in `tsconfig.json`.

### Testing

```bash
npm test         # Run Mocha tests with test environment
```

Tests are located in the `test/` directory and use Mocha + Chai. Test environment variables are loaded from `test/test.env`.

### Linting

```bash
npm run lint              # Run all linters
npm run lint:eslint       # Run ESLint on all files
npm run lint:markdown     # Run Markdownlint on Markdown files
npm run format            # Format code with Prettier
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neilenns/ambientweather2mqtt](https://github.com/neilenns/ambientweather2mqtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
