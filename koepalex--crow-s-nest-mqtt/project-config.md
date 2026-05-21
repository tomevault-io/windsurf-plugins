---
trigger: always_on
description: dotnet build CrowsNestMQTT.slnx --configuration Release
---

# Copilot Instructions for Crow's NestMQTT

## Build and Test Commands

### Build
```powershell
dotnet build CrowsNestMQTT.slnx --configuration Release
```

### Run All Tests
```powershell
dotnet test --configuration Release --collect:"XPlat Code Coverage" --settings coverlet.runsettings --filter "Category!=LocalOnly"
```

### Run Single Test Project
```powershell
# Unit tests
dotnet test tests/UnitTests/UnitTests.csproj

# Integration tests
dotnet test tests/integration/Integration.Tests.csproj

# Contract tests
dotnet test tests/contract/Contract.Tests.csproj
```

### Run the aspire environment
```powershell
dotnet run --project src/AppHost/AppHost.csproj --launch-profile http 
```

### Run Single Test
```powershell
dotnet test tests/UnitTests/UnitTests.csproj --filter "ClassName.TestMethodName"
```

### Generate Coverage Report
```powershell
dotnet test --no-build --configuration Release --collect:"XPlat Code Coverage" --settings coverlet.runsettings --results-directory ./TestResults
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator -reports:"./TestResults/**/coverage.cobertura.xml" -targetdir:"./TestResults/CoverageReport" -reporttypes:"Html;Cobertura;JsonSummary"
```

### Restore Dependencies
```powershell
dotnet restore
```

## Architecture Overview

### High-Level Structure
Crow's NestMQTT is a cross-platform MQTT GUI client built with **Avalonia** (cross-platform UI framework). The application follows a layered architecture with clear separation of concerns:

```
UI Layer (Avalonia)
    ↓
BusinessLogic Layer (MQTT handling, command processing)
    ↓
Utils Layer (Common utilities, shared models)
```

### Projects

- **CrowsNestMqtt.App** - Entry point; sets up DI, logging (Serilog), and initializes the Avalonia application
- **UI** - Presentation layer: Views, ViewModels, Commands, and UI Services using Avalonia/ReactiveUI
- **BusinessLogic** - Domain logic: MQTT engine, command parsing, message processing, export/import operations
- **Utils** - Shared utilities: buffer management, JSON tree building, logging helpers

### Key Components

#### MQTT Engine (BusinessLogic/MqttEngine.cs)
- Manages MQTT client lifecycle and reconnection logic
- Handles high-volume message streams with per-topic ring buffers
- Supports MQTT 5.0 features including enhanced authentication
- Manages message retention and correlation tracking
- `DefaultMaxTopicBufferSize = 1MB` (configurable per-topic)

#### Message Storage (Utils)
- `TopicRingBuffer` - Circular buffer for each topic with configurable size limits
- `BufferedMqttMessage` - Wraps MQTT messages with metadata and unique ID
- `TopicMessageStore` - Manages multi-level topic hierarchy with wildcard support

#### Command Processing (BusinessLogic/Commands)
- `CommandParserService` - Parses colon-prefixed commands (`:connect`, `:filter`, etc.)
- `CommandType` enum - Defines all supported commands
- Commands are executed through the UI Command Processor

#### UI Services (UI/Services)
- Dependency injection pattern for testability
- Services include: topic tree management, keyboard navigation, image/hex viewing, status bar updates
- ViewModels bind to services and expose reactive properties

### Configuration Management
- **Settings Storage** - `AppData\Local\CrowsNestMqtt\` (Serilog logs)
- **Settings Model** - `SettingsData` class with serialization support
- **Buffer Limits** - `TopicBufferLimit` for per-topic memory management
- **Connection Settings** - `MqttConnectionSettings` with TLS and auth mode support

## Command Interface Reference

Crow's NestMQTT is command-driven. Access the command palette with **Ctrl+Shift+P**. All commands use colon-prefixed syntax (`:command`). GUI elements support commands but do not replace them.

### Connection & Authentication Commands

- **`:connect [<server:port>]`** - Connect to MQTT broker. Use format `hostname:port` (e.g., `:connect mqtt.broker.com:1883`). If no arguments provided, uses connection settings from configuration.
- **`:disconnect`** - Disconnect from the current MQTT broker.
- **`:setuser <username>`** - Set MQTT authentication username.
- **`:setpass <password>`** - Set MQTT authentication password.
- **`:setauthmode <anonymous|userpass|enhanced>`** - Set authentication mode.
  - `anonymous` - No credentials required
  - `userpass` - Use username/password authentication
  - `enhanced` - MQTT 5.0 enhanced authentication with method-specific data
- **`:setauthmethod <method>`** - Set enhanced authentication method (e.g., `SCRAM-SHA-1`, `K8S-SAT`).
- **`:setauthdata <data>`** - Set enhanced authentication data (method-specific).
- **`:setusetls <true|false>`** - Enable/disable TLS encryption. When enabled, client allows untrusted certificates.

### Message Viewing & Filtering Commands

- **`:filter [regex_pattern]`** - Filter messages by regex pattern applied to payload. Omit pattern to clear filter.
- **`:view <raw|json|image|video|hex>`** - Switch payload viewer:
  - `raw` - Plain text view
  - `json` - Formatted JSON tree (when content-type is application/json)
  - `image` - Image viewer (when content-type indicates image)
  - `video` - Video player (when content-type indicates video)
  - `hex` - Hexadecimal viewer for binary data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koepalex/Crow-s-Nest-MQTT](https://github.com/koepalex/Crow-s-Nest-MQTT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
