---
trigger: always_on
description: This is a production-ready Chrome Extension boilerplate using Manifest V3, TypeScript, React, Vite, and Tailwind CSS.
---

# Chrome Extension Boilerplate - Development Guide

This is a production-ready Chrome Extension boilerplate using Manifest V3, TypeScript, React, Vite, and Tailwind CSS.

## Project Structure

```
├── src/
│   ├── background/index.ts    # Service worker - runs in background
│   ├── content/index.ts       # Content script - runs on web pages
│   ├── popup/                 # Popup UI (React) - toolbar icon click
│   ├── options/               # Options page (React) - extension settings
│   └── lib/
│       ├── storage.ts         # Type-safe Chrome storage
│       └── messaging.ts       # Type-safe message passing
├── public/icons/              # Extension icons (16, 32, 48, 128px)
├── manifest.json              # Chrome extension manifest (MV3)
└── vite.config.ts             # Build configuration
```

## Architecture Overview

### Communication Flow
```
┌─────────┐     messages      ┌────────────┐     messages      ┌─────────┐
│  Popup  │ ←───────────────→ │ Background │ ←───────────────→ │ Content │
│ (React) │                   │  (Worker)  │                   │ Script  │
└─────────┘                   └────────────┘                   └─────────┘
     ↓                              ↓                               ↓
 User clicks                  Handles all                    Runs on web
 toolbar icon                 Chrome APIs                    pages, can
                              and storage                    modify DOM
```

### Key Files to Modify

| Task | File(s) |
|------|---------|
| Add new setting | `src/lib/storage.ts` → StorageSchema |
| Add new message type | `src/lib/messaging.ts` → MessageTypes |
| Handle new message | `src/background/index.ts` → createMessageHandler |
| Modify popup UI | `src/popup/Popup.tsx` |
| Modify options page | `src/options/Options.tsx` |
| Add page manipulation | `src/content/index.ts` |
| Change permissions | `manifest.json` → permissions |

## Common Tasks

### Adding a New Storage Key

1. Edit `src/lib/storage.ts`:
```typescript
export interface StorageSchema {
  settings: { ... };
  userData: { ... };
  // Add your new key:
  myFeature: {
    enabled: boolean;
    data: string[];
  };
}
```

2. Use it anywhere:
```typescript
import { getStorage, setStorage } from "@/lib/storage";

const myData = await getStorage("myFeature");
await setStorage("myFeature", { enabled: true, data: ["item1"] });
```

### Adding a New Message Type

1. Edit `src/lib/messaging.ts`:
```typescript
export interface MessageTypes {
  // ... existing types ...

  FETCH_DATA: {
    request: { url: string };
    response: { data: unknown; success: boolean };
  };
}
```

2. Add handler in `src/background/index.ts`:
```typescript
createMessageHandler({
  // ... existing handlers ...

  FETCH_DATA: async (payload) => {
    const response = await fetch(payload.url);
    const data = await response.json();
    return { data, success: true };
  },
});
```

3. Call from popup/content:
```typescript
import { sendToBackground } from "@/lib/messaging";

const result = await sendToBackground("FETCH_DATA", { url: "https://api.example.com" });
if (result.success) {
  console.log(result.data);
}
```

### Adding a New Permission

Edit `manifest.json`:
```json
{
  "permissions": [
    "storage",
    "activeTab",
    "scripting",
    "tabs",          // Add for full tab access
    "notifications", // Add for notifications
    "alarms"         // Add for scheduled tasks
  ]
}
```

### Adding Keyboard Shortcuts

Edit `manifest.json`:
```json
{
  "commands": {
    "_execute_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "Command+Shift+Y"
      },
      "description": "Open extension popup"
    },
    "toggle-feature": {
      "suggested_key": {
        "default": "Ctrl+Shift+U"
      },
      "description": "Toggle feature"
    }
  }
}
```

Handle in background:
```typescript
chrome.commands.onCommand.addListener((command) => {
  if (command === "toggle-feature") {
    // Handle command
  }
});
```

### Adding Context Menu Items

In `src/background/index.ts`:
```typescript
chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: "my-action",
    title: "Do Something",
    contexts: ["selection", "page"],
  });
});

chrome.contextMenus.onClicked.addListener((info, tab) => {
  if (info.menuItemId === "my-action") {
    // Handle click
    // info.selectionText contains selected text
  }
});
```

### Injecting UI into Web Pages

In `src/content/index.ts`:
```typescript
function injectUI() {
  // Create shadow root to isolate styles
  const host = document.createElement("div");
  host.id = "my-extension-root";
  const shadow = host.attachShadow({ mode: "closed" });

  // Add your UI
  shadow.innerHTML = `
    <style>
      .container { /* styles isolated from page */ }
    </style>
    <div class="container">
      <button id="my-btn">Click me</button>
    </div>
  `;

  document.body.appendChild(host);

  // Add event listeners
  shadow.getElementById("my-btn")?.addEventListener("click", () => {
    // Handle click
  });
}
```

### Making API Calls

Create `src/lib/api.ts`:
```typescript
const API_BASE = "https://api.example.com";


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KenKaiii/kens-chrome-extension](https://github.com/KenKaiii/kens-chrome-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
