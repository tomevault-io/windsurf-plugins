---
trigger: always_on
description: This document provides context about the homebridge-mitsubishi-comfort plugin architecture, implementation details, and recent changes to help Claude (or other AI assistants) understand the codebase.
---

# Claude.md - Project Documentation for AI Assistance

This document provides context about the homebridge-mitsubishi-comfort plugin architecture, implementation details, and recent changes to help Claude (or other AI assistants) understand the codebase.

## Project Overview

This is a Homebridge plugin for Mitsubishi heat pumps using the Kumo Cloud v3 API. It provides HomeKit integration for controlling Mitsubishi mini-split systems.

**Current Version:** 1.7.0

## Architecture Overview

### Core Components

1. **platform.ts** - Main platform plugin
   - Handles device discovery and registration
   - Manages centralized site-level polling
   - Initializes streaming connection
   - Coordinates between accessories and API

2. **accessory.ts** - Individual thermostat accessory
   - Implements HomeKit thermostat service
   - Handles characteristic get/set operations
   - Receives updates from both streaming and polling
   - Manages device-specific state

3. **kumo-api.ts** - API client
   - Authentication with JWT tokens (auto-refresh every 15 minutes)
   - REST API endpoints for commands and device status
   - Socket.IO streaming for real-time updates
   - Connection management and error handling

4. **settings.ts** - Configuration and types
   - API endpoints and constants
   - TypeScript interfaces for all data structures
   - Configuration schema definitions

## Recent Major Changes

### v1.3.0 - Intelligent Streaming Health Monitoring and Adaptive Polling

**🎯 Goal:** Reduce API calls by 95% while maintaining reliability through smart fallback.

#### Key Achievement
- **Before:** ~257 API calls/hour (polling every 30s + streaming)
- **After:** ~12 API calls/hour (token refresh only when streaming healthy)
- **Reduction:** 95% fewer API calls and DNS queries

#### What Changed

**1. Streaming Health Monitoring (`kumo-api.ts`)**
- Added health tracking system that monitors Socket.IO connection status
- Health check every 30s (configurable)
- Callback system notifies platform of health changes
- Relies on Socket.IO's built-in heartbeat mechanism
- Code: `kumo-api.ts:36-42, 566-647`

**2. Adaptive Polling (`platform.ts`)**
- **Normal Mode:** Streaming healthy → polling disabled (if `disablePolling: true`)
- **Degraded Mode:** Streaming fails → fast polling activates (10s intervals)
- Automatic mode switching based on streaming health
- Comprehensive logging for all state transitions
- Code: `platform.ts:25-27, 343-458`

**3. Race Condition Prevention (`accessory.ts`)**
- Timestamp-based update filtering
- Prevents old polling data from overwriting newer streaming data
- Tracks update source (streaming vs polling)
- Code: `accessory.ts:15-16, 122-145`

**4. New Configuration Options**
- `disablePolling` - Now recommended! Enables optimal streaming-only mode
- `degradedPollInterval` - Fast polling when streaming unhealthy (default: 10s)
- `streamingHealthCheckInterval` - Health check frequency (default: 30s)
- `streamingStaleThreshold` - No longer used (deprecated, kept for compatibility)

#### How It Works

**Startup:**
```
1. Streaming connects → marked healthy
2. If disablePolling=true → no polling starts
3. Only token refresh queries (every 15 min)
```

**When Streaming Disconnects:**
```
1. Health check detects disconnect
2. Platform switches to DEGRADED MODE
3. Fast polling activates (10s intervals)
4. Devices remain responsive via polling
```

**When Streaming Reconnects:**
```
1. Socket reconnects → marked healthy
2. Platform switches to NORMAL MODE
3. Polling halts (if disablePolling=true)
4. Back to streaming-only updates
```

**Logging Examples:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Mitsubishi Comfort Plugin Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Streaming: ENABLED
Polling mode: On-demand only
Strategy: Streaming primary, polling fallback only
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Streaming connection established
Monitoring 3 device(s) for real-time updates

[When streaming fails]
✗ Streaming disconnected: transport close
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠ STREAMING INTERRUPTED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ Switching to DEGRADED MODE
→ Polling activated: 10s intervals
```

### v1.2.0 - Real-time Streaming Support

We added Socket.IO streaming to receive real-time device updates instead of relying solely on polling.

#### Implementation Details

**Streaming Connection:**
- Server: `socket-prod.kumocloud.com`
- Protocol: Socket.IO v4
- Transport: Polling initially, upgrades to WebSocket
- Authentication: Bearer token in extraHeaders

**Flow:**
1. Platform starts streaming after device discovery
2. Socket connects and emits 'subscribe' event for each device serial
3. Server sends 'device_update' events with full device state
4. Callbacks in accessory.ts process updates immediately
5. HomeKit characteristics update in real-time

**Key Code Locations:**
- Streaming initialization: `platform.ts:219-227`
- Socket.IO setup: `kumo-api.ts:418-497`
- Device subscription: `kumo-api.ts:499-507`
- Update handling: `accessory.ts:67-103`

#### Polling Strategy (Updated in v1.3.0)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [burtherman/homebridge-mitsubishi-comfort](https://github.com/burtherman/homebridge-mitsubishi-comfort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
