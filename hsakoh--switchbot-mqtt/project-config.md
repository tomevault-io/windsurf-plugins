---
trigger: always_on
description: This project is a **Home Assistant App (or add-on)** that bridges SwitchBot devices with MQTT, enabling integration with Home Assistant. It is a .NET 8 Blazor Server application that:
---

# GitHub Copilot Instructions for switchbot-mqtt

## Project Overview

This project is a **Home Assistant App (or add-on)** that bridges SwitchBot devices with MQTT, enabling integration with Home Assistant. It is a .NET 8 Blazor Server application that:

- Communicates with SwitchBot Cloud API to control physical and virtual infrared remote devices
- Publishes device states and accepts commands via MQTT protocol
- Provides a web UI for device configuration and management
- Supports both polling and webhook-based device state updates
- Integrates with Home Assistant MQTT Discovery for automatic device setup

## Architecture

### Projects Structure

1. **SwitchBotMqttApp** (`src/SwitchBotMqttApp/`)
   - Main Blazor Server application
   - Ports: 8098 (Webhook), 8099 (Web UI)
   - Contains device management logic, MQTT services, and web UI

2. **HomeAssistantAddOn.Mqtt** (`src/HomeAssistant/HomeAssistantAddOn.Mqtt/`)
   - MQTT client library for Home Assistant integration
   - Handles MQTT broker connection and message publishing

3. **HomeAssistantAddOn.Core** (`src/HomeAssistant/HomeAssistantAddOn.Core/`)
   - Core utilities for Home Assistant Add-on functionality
   - Configuration management

### Key Components

#### Services
- **MqttCoreService**: Manages MQTT connections and message publishing/subscribing
- **PollingService**: Polls SwitchBot API periodically to get device status updates
- **WebhookService**: Handles webhook registration and real-time device event updates
- **AutomatedHostedService**: Coordinates service lifecycle and startup sequence

#### Managers
- **DeviceConfigurationManager**: Manages device configurations (physical and virtual devices)
- **DeviceDefinitionsManager**: Loads and manages device type definitions and capabilities
- **DeviceStatePersistanceManager**: Persists device state across application restarts

#### API Client
- **SwitchBotApiClient**: Handles authentication and communication with SwitchBot Cloud API (v1.1)

## Technology Stack

- **.NET 8**: Target framework
- **Blazor Server**: Interactive web UI with SignalR
- **MQTTnet**: MQTT client library
- **Blazored.Modal**: Modal dialog support
- **FluffySpoon.Ngrok**: Ngrok integration for webhook tunneling
- **Home Assistant MQTT Discovery**: Auto-configuration protocol

## Coding Guidelines

### General Principles

1. **Follow existing patterns**: This codebase has established patterns for services, managers, and models
2. **Async/await**: All I/O operations should be asynchronous with proper CancellationToken support
3. **Dependency Injection**: Use constructor injection for all dependencies
4. **Logging**: Use structured logging with appropriate log levels
5. **Configuration**: Use Options pattern for configuration management

### Naming Conventions

- **Services**: Suffix with `Service` (e.g., `MqttCoreService`, `PollingService`)
- **Managers**: Suffix with `Manager` (e.g., `DeviceConfigurationManager`)
- **Options**: Suffix with `Options` (e.g., `SwitchBotOptions`, `WebhookServiceOptions`)
- **Models**: Use descriptive names without suffixes unless needed for clarity
- **MQTT entities**: Suffix with `Config` (e.g., `ButtonConfig`, `SensorConfig`)

### Project-Specific Patterns

#### Configuration Management
```csharp
// Use IOptions<T> pattern for configuration
public class MyService
{
    private readonly IOptions<MyOptions> _options;
    
    public MyService(IOptions<MyOptions> options)
    {
        _options = options;
    }
}
```

#### Device Models
- **PhysicalDevice**: Represents actual SwitchBot hardware
- **VirtualInfraredRemoteDevice**: Represents IR remote controls
- Device definitions are loaded from JSON files in device-definitions directory

#### MQTT Message Patterns
- Config topic: `homeassistant/{component}/{objectId}/config`
- Command topic: `switchbot-mqtt/{deviceId}/command`
- State topic: `switchbot-mqtt/{deviceId}/state`
- Use Home Assistant MQTT Discovery format

#### API Rate Limiting
- SwitchBot API has rate limits (10,000 calls/day)
- Track API call counts with `ApiCallCount` dictionary
- Prefer webhook updates over polling when available

### Error Handling

1. **API Errors**: Handle SwitchBot API status codes appropriately
2. **MQTT Errors**: Implement reconnection logic
3. **Logging**: Log exceptions with context (device ID, operation type, etc.)
4. **User Feedback**: Show meaningful error messages in the UI

### Testing Considerations

1. **Mock SwitchBot API**: Use HttpClient factory for testability
2. **Debug proxy**: Development mode includes Fiddler proxy support (localhost:8888)
3. **Ngrok support**: For webhook testing in development environment

## Device Support

### Physical Devices
Support for SwitchBot devices including:
- Bot, Curtain, Hub, Plug, Lock
- Sensors (Motion, Contact, Meter, Indoor/Outdoor Meter)
- Lights (Color Bulb, Strip Light, RGBICWW variants)
- Humidifiers, Fans, Blinds, etc.

### Virtual Infrared Remote Devices
Support for IR remotes controlling:
- Air Conditioners, TVs, Lights, Fans
- Projectors, Speakers, Cameras
- And more (see device-definitions JSON files)

## Home Assistant Integration

### MQTT Discovery

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hsakoh/switchbot-mqtt](https://github.com/hsakoh/switchbot-mqtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
