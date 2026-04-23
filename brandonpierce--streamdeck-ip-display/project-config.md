---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Stream Deck plugin that displays IP addresses on Stream Deck buttons with configurable auto-refresh functionality. It offers four button types: Dual IP Display (both local and public), Local IP Only, Public IP Only, and IP Toggle Display (cycles between modes). Features Canvas-based rendering for pixel-perfect display, WiFi SSID display, and user-configurable refresh intervals via property inspector UI.

## Development Commands

```bash
# Build the plugin
npm run build

# Build and watch for changes (auto-restarts Stream Deck plugin)
npm run watch

# Manual Stream Deck operations (requires Elgato CLI)
streamdeck restart io.piercefamily.ip-display
streamdeck link
streamdeck dev
```

## Architecture

### Plugin Structure
- **`src/plugin.ts`** - Main plugin entry point that registers all actions and connects to Stream Deck
- **`src/actions/local-ip-display.ts`** - Dual IP display action (shows both local and public IP)
- **`src/actions/local-ip-only-display.ts`** - Local IP only display action
- **`src/actions/public-ip-only-display.ts`** - Public IP only display action
- **`src/actions/ip-toggle-display.ts`** - Toggle IP display action (cycles between modes)
- **`io.piercefamily.ip-display.sdPlugin/`** - Stream Deck plugin directory with manifest, assets, and built code
- **`io.piercefamily.ip-display.sdPlugin/ui/`** - Property inspector HTML files for settings configuration

### Key Patterns

**Action Registration**: Uses `@action()` decorator with UUID matching manifest.json:
```typescript
@action({ UUID: "io.piercefamily.ip-display.dual-ip" })
export class IPDisplay extends SingletonAction<IPSettings>
```

**Canvas Rendering**: All visual output uses Canvas instead of text to avoid truncation issues:
- 144x144 pixel resolution for Stream Deck buttons
- Text shadows for readability on transparent backgrounds
- Base64 data URI conversion for Stream Deck compatibility

**IP Address Handling**:
- Local IP: Uses `os.networkInterfaces()` to find non-internal IPv4 addresses
  - Supports user-selected network interface via dropdown (auto-detect default)
  - Settings persist with `networkInterface?: string` field
- Public IP: Fetches from ipify.org API with 5-minute caching to avoid rate limits
- Status indicator: Color-coded dot (green/orange/red) based on connection status

**WiFi SSID Detection**:
- Uses native OS commands via `child_process.exec()` with `promisify` wrapper
- Platform-specific command detection:
  - Windows: `netsh wlan show interfaces` (regex: `/^\s*SSID\s*:\s*(.+)$/m`)
  - macOS: `system_profiler SPAirPortDataType` (regex: `/Current Network Information:[\s\S]*?\n\s+([^:]+):/`)
  - Linux/other: Returns `null` (not supported)
- 5-minute caching (`SSID_CACHE_DURATION = 5 * 60 * 1000`) with timestamp tracking
- Timeout: 5 seconds per command execution
- Fails silently if command fails (Ethernet, disconnected, or unsupported platform)
- Cache structure: `{ ssid: string | null; timestamp: number }`
- Only displayed for local IP (not public IP) when `showWifiSSID !== false` (default true)
- Canvas rendering: Gray text (#999999) below LOCAL IP label
  - Font: 10-11px Arial (varies by mode)
  - Truncation: 15-20 chars depending on layout with "..." suffix
  - Position: 12-15px below label text

**Clipboard Copy Pattern**:
- Long-press detection using 800ms threshold (`LONG_PRESS_THRESHOLD`)
- `onKeyDown`: Starts timer, stores press timestamp and IPs
- `onKeyUp`: Checks duration, executes copy if ≥ threshold, else refreshes
- Uses `clipboardy` library for cross-platform clipboard access
- Visual feedback via `showOk()` (success) or `showAlert()` (failure)
- Dual IP format: `<localIP>,<publicIP>` (comma-separated)
- Single IP format: Just the IP address without any formatting

**Custom Label Support**:
- Settings fields: `customLabel`, `customLocalLabel`, `customPublicLabel`
- Max 12 characters to fit button layout
- Falls back to defaults ("LOCAL IP", "PUBLIC IP") when blank
- Renders using Canvas with dynamic font sizing

**Multi-line Display Mode**:
- Toggle via `multilineIP?: boolean` setting
- Splits IP addresses into two lines: `192.168.` / `1.100`
- Larger fonts (20px vs 16px for IPs, 13px vs 14px for labels)
- Different vertical spacing calculations for single vs multi-line
- `splitIP()` utility function handles formatting

**Custom Color Customization**:
- Settings fields: `labelColor?: string`, `ipColor?: string`
- Color picker UI using `<sdpi-color>` SDPI Components
- Canvas rendering uses fallback pattern: `settings.labelColor || '#C0C0C0'`
- Defaults: Silver (#C0C0C0) for labels, White (#FFFFFF) for IP addresses
- All color pickers implemented in all four action types
- Optional fields maintain backward compatibility
- Status dots remain hardcoded (green/orange/red) for connection status

**Status Dot Behavior**:
- Positioned inline with labels (to the left) using dynamic text measurement
- Single IP displays: Green (#00FF00) if connected, Red (#FF6B6B) if disconnected
- Dual IP displays: Green (#00FF00) both connected, Orange (#FFAA00) partial, Red (#FF6B6B) none

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brandonpierce) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
