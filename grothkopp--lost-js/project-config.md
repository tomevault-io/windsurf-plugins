---
trigger: always_on
description: When asked to create a new application using the LOST Framework (Local, Offline, Shareable Tools), follow these guidelines to ensure consistency, performance, and full feature utilization.
---

# Instructions for AI Agents: Creating a LOST App

When asked to create a new application using the LOST Framework (Local, Offline, Shareable Tools), follow these guidelines to ensure consistency, performance, and full feature utilization.

## 1. Project Structure & Naming

All LOST apps are single-page applications (SPAs) served from a subfolder or root.

**Naming Convention:**
- App Name: `[app-name]` (e.g., `counter`, `todo`, `dice`)
- HTML File: `[app-name].html`
- JS File: `app_[app-name].js`
- CSS File: `app_[app-name].css`
- Manifest: `[app-name].webmanifest`
- Assets: `[app-name]-icon-192.png`, etc.

**Directory Layout:**
```text
/
  lost.js       (Core Framework)
  lost-ui.js    (UI Shell)
  lost.css      (UI Styles)
  [app-name]/
    [app-name].html
    app_[app-name].js
    app_[app-name].css
    [app-name].webmanifest
    icon.png
```

---

## 2. The Base HTML (`[app-name].html`)

The HTML should be minimal. It must import `lost.css`, your app's CSS, and the frameworks as modules.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta name="theme-color" content="#ffffff">
    <title>App Name</title>
    
    <!-- Manifest for PWA -->
    <link rel="manifest" href="[app-name].webmanifest">
    
    <!-- Framework Styles -->
    <link rel="stylesheet" href="/lost.css">
    <!-- App Styles -->
    <link rel="stylesheet" href="app_[app-name].css">
</head>
<body>
    <!-- App Specific Content Area -->
    <main id="app-stage">
        <!-- Dynamic content goes here -->
        <div id="output">0</div>
        <button id="incrementBtn">Count</button>
    </main>

    <!-- Configuration Dialog (Standard Pattern) -->
    <dialog id="configDialog">
        <div class="dialog-header">Settings</div>
        <div class="dialog-body">
            <label>
                Title:
                <input type="text" id="configTitle">
            </label>
        </div>
        <div class="dialog-footer">
            <button id="configCloseBtn">Close</button>
        </div>
    </dialog>

    <!-- Framework Modules -->
    <script type="module" src="/lost.js"></script>
    <script type="module" src="/lost-ui.js"></script>
    <script type="module" src="app_[app-name].js"></script>
</body>
</html>
```

---

## 3. The Application Logic (`app_[app-name].js`)

The entry point should be a class that initializes `Lost` and `LostUI`.

### Step 1: Define Defaults
```javascript
import { Lost } from '/lost.js';
import { LostUI } from '/lost-ui.js';

const DEFAULT_DATA = {
    title: 'New Item',
    count: 0
};
```

### Step 2: Initialize Class
```javascript
class CounterApp {
    constructor() {
        // 1. Setup Data Layer
        this.lost = new Lost({
            storageKey: 'app-counter-v1',
            defaultData: DEFAULT_DATA,
            // Validate incoming data to prevent corruption
            validator: (data) => typeof data.count === 'number'
        });

        // 2. Listen for State Changes
        this.lost.addEventListener('update', (e) => this.render(e.detail));

        // 3. Setup UI Shell
        this.ui = new LostUI(this.lost, {
            container: document.body,
            header: {
                title: 'Counter App',
                // Add a settings button to the header
                extraContent: () => {
                    const btn = document.createElement('button');
                    btn.innerHTML = '⚙️';
                    btn.onclick = () => this.openConfig();
                    return btn;
                }
            },
            sidebar: {
                heading: 'Counters',
                onNew: () => this.createItem(),
                // Custom list item display
                title: (item) => item.title || 'Untitled',
                subline: (item) => `Current count: ${item.count}`
            }
        });

        // 4. Bind App Elements
        this.elements = {
            output: document.getElementById('output'),
            btn: document.getElementById('incrementBtn'),
            dialog: document.getElementById('configDialog'),
            configTitle: document.getElementById('configTitle'),
            closeDialog: document.getElementById('configCloseBtn')
        };

        this.bindEvents();
        this.init();
    }

    async init() {
        this.lost.load(); // Load data
        this.ui.load();   // Init UI
    }
```

### Step 3: Events & State Updates
**CRITICAL**: Never modify `this.lost.getCurrent()` directly. Always use `this.lost.update()`.

```javascript
    bindEvents() {
        // App Action
        this.elements.btn.addEventListener('click', () => {
            const item = this.lost.getCurrent();
            if (item) {
                this.lost.update(item.id, { count: item.count + 1 });
            }
        });

        // Config Dialog Handling
        this.elements.closeDialog.addEventListener('click', () => {
            this.elements.dialog.close();
        });

        // Live Update from Config
        this.elements.configTitle.addEventListener('input', (e) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grothkopp/lost.js](https://github.com/grothkopp/lost.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
