---
trigger: always_on
description: Control React Native mobile apps via CLI commands. Discover UI elements, tap buttons, type text, toggle switches, scroll views, and navigate screens programmatically. Run natural language test cases autonomously.
---


# React Native AgentKit — AI Agent Skill

You can control React Native mobile apps that have integrated `react-native-agentkit`. The app exposes its live UI as structured data over a WebSocket connection, and you interact with it by sending JSON commands.

## Core Loop

Your workflow follows an **observe → reason → act** cycle:

1. **Observe** — Send `state` or `list` to see all interactive elements on screen
2. **Reason** — Map the user's intent to specific element IDs and actions
3. **Act** — Send commands (`tap`, `type`, `scroll`, etc.) to interact
4. **Repeat** — Each response includes updated screen state; continue until the task is done

## Pre-Flight Check

**Before interacting with the app UI, always check if the app is already connected.** If it is, skip device setup entirely:

```bash
# Step 0: Check if app is already connected
echo '{"cmd":"ping"}' | npx react-native-agentkit pipe --relay=ws://localhost:8347
```

- If you get `{"success": true, "result": {"pong": true}}` → **skip to UI commands** (the app is running)
- If you get an error or timeout → **proceed with device setup** below

This prevents unnecessary device operations when the app is already running and connected.

## Device Setup (when app is not running)

If the pre-flight check fails, you can autonomously prepare the device. Device commands do NOT require a relay connection — they operate directly on the OS.

### Setup Flow

```bash
# 1. Check what's available
npx react-native-agentkit device list --platform=ios

# 2. Boot a simulator (fuzzy-matches by name)
npx react-native-agentkit device boot --name="iPhone 16"

# 3. Pre-grant permissions (prevents native alert interruptions during automation)
npx react-native-agentkit device grant-permissions --bundleId=com.myapp --permissions=camera,photos,location,notifications

# 4. Install the app
npx react-native-agentkit device install ./ios/build/Build/Products/Debug-iphonesimulator/MyApp.app

# 5. Launch the app
npx react-native-agentkit device launch com.myapp

# 6. Wait a moment for relay connection, then begin UI automation
echo '{"cmd":"state"}' | npx react-native-agentkit pipe --relay=ws://localhost:8347
```

### Device Command Reference

| Command | Purpose | Example |
| --- | --- | --- |
| `device list` | List simulators/emulators | `device list --platform=ios` |
| `device boot` | Boot a device (fuzzy name match) | `device boot --name="iPhone 16"` |
| `device shutdown` | Shutdown a device | `device shutdown <udid>` |
| `device install` | Install app | `device install ./path/to/MyApp.app` |
| `device uninstall` | Uninstall app | `device uninstall com.myapp` |
| `device launch` | Launch an app | `device launch com.myapp` |
| `device terminate` | Terminate a running app | `device terminate com.myapp` |
| `device erase` | Factory reset a device | `device erase <udid>` |
| `device create-emulator` | Create Android AVD | `device create-emulator MyDevice` |
| `device grant-permissions` | Pre-grant OS permissions | `device grant-permissions --bundleId=com.myapp --permissions=camera,photos` |
| `device reset-permissions` | Reset permissions to prompt | `device reset-permissions --bundleId=com.myapp` |
| `device clear-data` | Clear app data (Android) | `device clear-data com.myapp` |
| `device status` | Show all devices and tools | `device status` |

All device commands output JSON and can be used in automation scripts.

### Permission Pre-Granting

Pre-granting OS permissions **before** launching the app prevents native permission dialogs from appearing during automation. This is critical for uninterrupted test flows.

**iOS** — available services:
`calendar`, `contacts`, `location`, `location-always`, `camera`, `microphone`, `photos`, `reminders`, `siri`, `notifications`, `speech-recognition`, `motion`, `media-library`, or `all`

```bash
npx react-native-agentkit device grant-permissions \
  --bundleId=com.myapp --permissions=camera,photos,location,notifications
```

**Android** — uses short or full permission names:
```bash
npx react-native-agentkit device grant-permissions \
  --packageId=com.myapp --permissions=CAMERA,ACCESS_FINE_LOCATION
```

> **Note:** On Android, `device install` already grants all manifest-declared permissions by default (via `adb install -g`). Use `grant-permissions` only for selective granting after install.

> **Note:** Permission pre-granting handles **OS-level** permission dialogs. The app's built-in `NativeDialogInterceptor` handles **JS-level** dialogs (Alert.alert, react-native-permissions). Both layers work together — pre-grant to prevent OS dialogs, and the interceptor catches any remaining JS-level ones.

### Physical iOS Device

If the user asks to run on a physical iPhone/iPad, use `ios-deploy` (auto-installed via Homebrew if missing):

```bash
# List connected physical devices
npx react-native-agentkit device list --platform=ios
# (Physical devices appear alongside simulators)

# Install and launch on physical device
npx react-native-agentkit device install ./MyApp.app --device=<udid>
npx react-native-agentkit device launch com.myapp --device=<udid>
```

### Expo Projects


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junocs/react-native-agentkit](https://github.com/junocs/react-native-agentkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
