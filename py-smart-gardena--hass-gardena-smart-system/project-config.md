---
trigger: always_on
description: This is a Home Assistant custom integration for Gardena Smart System devices. The integration provides real-time control and monitoring of Gardena devices (lawn mowers, valves, sensors, power sockets) through the Gardena Smart System API v2.
---

# Gardena Smart System Integration for Home Assistant - Development Rules

## 🎯 Project Overview
This is a Home Assistant custom integration for Gardena Smart System devices. The integration provides real-time control and monitoring of Gardena devices (lawn mowers, valves, sensors, power sockets) through the Gardena Smart System API v2.

## 🏗️ Architecture

### Core Components
- **Authentication**: OAuth2 with Husqvarna/Gardena API
- **API Client**: REST API client for initial data loading
- **WebSocket Client**: Real-time event handling
- **Coordinator**: Data management and entity updates
- **Entity Platforms**: Home Assistant entities for each device type

### Key Files
- `custom_components/gardena_smart_system/__init__.py` - Main entry point
- `custom_components/gardena_smart_system/coordinator.py` - Data coordinator
- `custom_components/gardena_smart_system/gardena_client.py` - API client
- `custom_components/gardena_smart_system/websocket_client.py` - WebSocket client
- `custom_components/gardena_smart_system/auth.py` - Authentication manager
- `custom_components/gardena_smart_system/models.py` - Data models and parser

## 🔧 Development Environment

### Setup Commands
```bash
make setup          # Setup complete development environment
make ha-start       # Start Home Assistant with integration
make ha-stop        # Stop Home Assistant
make ha-logs        # View Home Assistant logs
make ha-reset       # Reset Home Assistant environment
make test           # Run all tests
make test-auth      # Run authentication tests only
```

### Environment Variables
- `GARDENA_CLIENT_ID` - Your Gardena Client ID
- `GARDENA_CLIENT_SECRET` - Your Gardena Client Secret
- `GARDENA_API_KEY` - Your Gardena API Key treived from authentification

## 📡 API Optimization Strategy

### Current Implementation (Optimized)
The integration now uses a **minimal API approach** to avoid rate limiting (429 errors):

1. **Authentication** (1 call at startup)
   - OAuth2 token request to Husqvarna authentication server
   - Automatic token refresh when needed

2. **Initial Data Loading** (1 call at startup)
   - Single call to `/locations` to get all locations and devices
   - Data stored in memory, no periodic reloading

3. **WebSocket Connection** (persistent)
   - Real-time updates via WebSocket
   - Automatic reconnection on disconnection
   - All device state changes come through WebSocket

### API Calls Reduction
- **Before**: ~1442 calls per day (polling every 60s)
- **After**: 2 calls per day (startup only)
- **Reduction**: 99.86% fewer API calls

## 🏠 Home Assistant Integration

### Supported Platforms
- `lawn_mower` - Gardena lawn mowers (SILENO, etc.)
- `valve` - Irrigation control valves
- `switch` - Power sockets
- `sensor` - Environmental sensors (temperature, humidity, light)
- `binary_sensor` - Battery status sensors

### Entity Structure
Each device can have multiple services:
- `COMMON` - Device information, battery, RF link status
- `MOWER` - Mower-specific data (activity, operating hours)
- `VALVE` - Valve control and status
- `POWER_SOCKET` - Power socket control
- `SENSOR` - Sensor readings

### Data Models
- `GardenaLocation` - Location with devices
- `GardenaDevice` - Device with multiple services
- `GardenaService` - Service-specific data (Mower, Valve, etc.)

## 🔐 Authentication

### OAuth2 Flow Implementation
The integration uses **Client Credentials Grant** OAuth2 flow with the Husqvarna/Gardena authentication system:

1. **Initial Authentication**:
   - `POST /v1/oauth2/token` with `grant_type=client_credentials`
   - Requires `client_id` and `client_secret`
   - Returns `access_token` and `expires_in`

2. **Token Management**:
   - Automatic token validation before each request
   - Token refresh 5 minutes before expiration
   - Thread-safe authentication with async locks

3. **SSL Context Handling**:
   - Cached SSL context for performance
   - Development mode bypass for macOS SSL issues
   - Proper SSL context management in async environment

### Authentication Headers
All API requests require these headers:
```python
{
    "Authorization": "Bearer <access_token>",
    "Authorization-Provider": "husqvarna",
    "X-Api-Key": "<client_id>",
    "Content-Type": "application/vnd.api+json"
}

```

### Error Handling
- **401 Unauthorized**: Token expired or invalid
- **403 Forbidden**: Insufficient permissions
- **429 Too Many Requests**: Rate limiting (handled by optimization)
- **Network Errors**: Automatic retry with exponential backoff

### Security Best Practices
- Tokens stored in memory only (not persisted)
- Automatic token invalidation on refresh failure
- Secure token expiration handling
- No sensitive data in logs
- SSL context caching for performance
- Proper async/await patterns for authentication
- Thread-safe token management with locks

### Authentication Implementation Details
Based on the SmartSystem library patterns:

1. **SSL Context Management**:
   ```python
   @functools.lru_cache(maxsize=1)
   def get_ssl_context():
       context = ssl.create_default_context()
       return context
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [py-smart-gardena/hass-gardena-smart-system](https://github.com/py-smart-gardena/hass-gardena-smart-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
