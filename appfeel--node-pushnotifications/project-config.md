---
trigger: always_on
description: This repository implements a Node.js module for sending push notifications across multiple platforms: Apple (APN), Google (FCM), Windows (WNS), Amazon (ADM), and Web-Push. The core logic is in `lib/` and `src/`, with each platform handled by a dedicated file (e.g., `sendAPN.js`, `sendFCM.js`).
---

# Copilot Instructions for AI Agents

## Project Overview

This repository implements a Node.js module for sending push notifications across multiple platforms: Apple (APN), Google (FCM), Windows (WNS), Amazon (ADM), and Web-Push. The core logic is in `lib/` and `src/`, with each platform handled by a dedicated file (e.g., `sendAPN.js`, `sendFCM.js`).

**Note:** Legacy GCM (Google Cloud Messaging) support has been removed. All Android push notifications now route exclusively through Firebase Cloud Messaging (FCM) using the Firebase Admin SDK.

## Architecture & Data Flow

- **Entry Point:** Use `PushNotifications` from `lib/index.js` or `src/index.js`.
- **Settings:** Each platform's credentials/config are passed to the constructor. See the example in `README.md`.
- **Sending:** The main method is `push.send(registrationIds, data, callback)` or as a Promise. It auto-detects device type and routes to the correct sender.
- **Platform Senders:** Each sender (e.g., `sendAPN.js`, `sendFCM.js`) implements the logic for its platform. Shared utilities are in `lib/utils/` and `src/utils/`.
- **RegId Detection:** Device type is inferred from the registration ID structure. See `PN.getPushMethodByRegId` for details.

## Developer Workflows

- **Install:** `npm install` (requires Node.js 28+)
- **Test:** Run all tests with `npm test`. Tests are in `test/` and cover basic flows and platform-specific cases (87 tests, all passing).
- **Debug:** Use the callback or Promise error/result from `push.send`. Each result object includes method, success/failure counts, and error details per device.
- **Build:** No build step required for basic usage. ES6 is used, but compatible with ES5 via Babel if needed.

## Conventions & Patterns

- **Platform-specific files:** Each push service has its own file for isolation and clarity. Legacy GCM is no longer supported.
- **Unified Data Model:** The `data` object for notifications is normalized across platforms. See `README.md` for all supported fields.
- **Error Handling:** Errors are unified and returned in the result array from `push.send`.
- **RegId Format:** Prefer object format for registration IDs (`{id, type}`), but string format is supported for legacy reasons.
- **Android Routing:** All Android push notifications route through FCM using Firebase Admin SDK.
- **Chunking:** Android tokens are chunked in batches of 1,000 automatically by FCM.
- **Constants:** Use constants from `constants.js` for platform types. Available constants: `FCM_METHOD`, `APN_METHOD`, `WNS_METHOD`, `ADM_METHOD`, `WEB_METHOD`, `UNKNOWN_METHOD`.

## Firebase Cloud Messaging (FCM) - Modern Implementation

### Message Building (src/utils/tools.js)

**buildAndroidMessage(data, options)**
- Converts unified notification data to Firebase Admin SDK AndroidMessage format
- Returns plain JavaScript object (no wrapper functions)
- Properties mapped to camelCase (Firebase SDK standard)
- Removes undefined values for clean API calls
- Converts TTL from seconds to milliseconds
- Supports all 20+ AndroidNotification properties

**buildAndroidNotification(data)**
- Maps input `data` object to AndroidNotification interface
- Supported properties:
  - Basic: `title`, `body`, `icon`, `color`, `sound`, `tag`, `imageUrl`
  - Localization: `titleLocKey`, `titleLocArgs`, `bodyLocKey`, `bodyLocArgs`
  - Android-specific: `channelId`, `notificationCount`, `ticker`, `sticky`, `visibility`
  - Behavior: `clickAction`, `priority`, `localOnly`, `eventTimestamp`
  - Accessibility: `ticker`
  - Vibration: `vibrateTimingsMillis`, `defaultVibrateTimings`
  - Sound: `defaultSound`
  - LED: `lightSettings`, `defaultLightSettings`
  - Proxy: `proxy` (notification-level, values: 'allow', 'deny', 'if_priority_lowered')

### FCM Configuration (src/sendFCM.js)

**Initialization Options:**
- `credential` or `serviceAccountKey` (required) - Firebase authentication
- `projectId` (optional) - Explicit Google Cloud project ID
- `databaseURL` (optional) - Realtime Database URL
- `storageBucket` (optional) - Cloud Storage bucket
- `serviceAccountId` (optional) - Service account email
- `databaseAuthVariableOverride` (optional) - Auth override for RTDB rules
- `httpAgent` (optional) - HTTP proxy agent for network requests
- `httpsAgent` (optional) - HTTPS proxy agent for network requests

All optional properties are dynamically added to Firebase initialization if defined.

### Proxy Support

**Two levels of proxy configuration:**

1. **Network-level (SDK initialization)**
   - `settings.fcm.httpAgent` and `settings.fcm.httpsAgent`
   - Controls how Firebase Admin SDK communicates with Google servers
   - Uses proxy agent libraries (http-proxy-agent, https-proxy-agent)
   - Applied at app initialization

2. **Notification-level (Android device)**
   - `data.proxy` property in notification message
   - Controls how Android devices handle notifications in proxy scenarios
   - Values: 'allow', 'deny', 'if_priority_lowered'
   - Per-message configuration

### Message Format

Firebase Admin SDK expects:
```javascript
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appfeel/node-pushnotifications](https://github.com/appfeel/node-pushnotifications) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
