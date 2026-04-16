---
trigger: always_on
description: Web Vibes is a Chrome extension that allows users to create and apply custom CSS/JavaScript "vibes" (hacks) to websites. The extension features a modern popup interface with gradient themes and a comprehensive settings system.
---

# Web Vibes Chrome Extension - Copilot Instructions

## Project Overview

Web Vibes is a Chrome extension that allows users to create and apply custom CSS/JavaScript "vibes" (hacks) to websites. The extension features a modern popup interface with gradient themes and a comprehensive settings system.

## Architecture Patterns

### 1. Three-Layer Architecture (Model → Repository → Service)

**ALL data management MUST follow this pattern:**

```
lib/[domain]/
├── model/
│   └── [domain].js          # Data model class
├── repo/
│   └── [domain]-repo.js     # Repository for storage operations
└── service/
    └── [domain]-service.js  # Business logic service
```

#### Model Layer Responsibilities:

- Data structure definition
- Validation methods (`isValid()`)
- Serialization (`toJSON()`, `fromJSON()`)
- Default values (`getDefaults()`)
- Static utility methods

#### Repository Layer Responsibilities:

- Chrome storage API interactions
- Storage key management
- Error handling for storage operations
- Storage usage/quota information

#### Service Layer Responsibilities:

- Business logic operations
- Data transformations
- Combining model and repository operations
- Takes repository as constructor dependency

### 2. Example Implementation Pattern

```javascript
// Model
class Settings {
  static getDefaults() {
    /* ... */
  }
  static fromJSON(data) {
    /* ... */
  }
  toJSON() {
    /* ... */
  }
  static isValid(data) {
    /* ... */
  }
}

// Repository
class SettingsRepository {
  constructor() {
    this.storageKey = "webVibesSettings";
  }
  async getSettings() {
    /* uses chrome.storage */
  }
  async saveSettings(settings) {
    /* uses chrome.storage */
  }
}

// Service
class SettingsService {
  constructor(settingsRepository) {
    this.repository = settingsRepository;
  }
  async getAllSettings() {
    return this.repository.getSettings();
  }
  async setTheme(themeKey) {
    /* business logic */
  }
}

// Usage in apps
class App {
  constructor() {
    this.repository = new SettingsRepository();
    this.service = new SettingsService(this.repository);
  }
}
```

## CSS Organization

### 1. CSS Import Hierarchy

**ALL HTML pages MUST follow this import pattern:**

```html
<head>
  <!-- Always import main.css first -->
  <link rel="stylesheet" href="main.css" />
  <!-- For root pages -->
  <link rel="stylesheet" href="../main.css" />
  <!-- For subdirectory pages -->
  <!-- Then page-specific styles -->
  <link rel="stylesheet" href="page-specific.css" />
</head>
```

### 2. main.css Structure

```css
/* External dependencies (Material Icons, fonts) */
@import url("...");

/* Base styles and CSS variables */
@import url("popup.css");

/* Global utilities and Material Icons setup */
.material-icons {
  /* proper rendering setup */
}
```

### 3. Theme System

**Dynamic theming MUST use CSS custom properties:**

```css
/* In main.css */
:root {
  --theme-gradient: linear-gradient(...); /* default */
}

.theme-cosmic-purple {
  --theme-gradient: linear-gradient(...);
}
.theme-sunset-glow {
  --theme-gradient: linear-gradient(...);
}
/* etc for all themes */

/* In component styles */
.header {
  background: var(--theme-gradient);
}
```

## File Organization

### 1. Directory Structure

```
popup/
├── main.css              # Central CSS imports
├── popup.css             # Core styles and variables
├── popup.html            # Main popup page
├── popup.js              # Main popup logic
├── CSS_README.md         # CSS documentation
└── settings/
    ├── settings.css      # Settings-specific styles
    ├── settings.html     # Settings page
    └── settings.js       # Settings page logic

lib/
├── index.js              # Library entry point
├── hack/                 # Hack domain
│   ├── model/hack.js
│   ├── repo/hack-repo.js
│   └── service/hack-service.js
└── settings/             # Settings domain
    ├── model/settings.js
    ├── repo/settings-repo.js
    ├── service/settings-service.js
    └── README.md
```

### 2. HTML Script Loading Order

**ALWAYS load in dependency order:**

```html
<!-- Model classes first -->
<script src="../lib/[domain]/model/[domain].js"></script>
<!-- Repository classes second -->
<script src="../lib/[domain]/repo/[domain]-repo.js"></script>
<!-- Service classes third -->
<script src="../lib/[domain]/service/[domain]-service.js"></script>
<!-- Application logic last -->
<script src="app.js"></script>
```

## UI/UX Standards

### 1. Material Icons Usage

**ALWAYS use Material Icons for interactive elements:**

```html
<!-- Settings button -->
<span class="material-icons">settings</span>
<!-- Back button -->
<span class="material-icons">arrow_back</span>
<!-- Checkmarks -->
<span class="material-icons">check</span>
```

### 2. Theme Implementation

**Gradient themes MUST include:**

- Name (display name)
- Gradient (CSS linear-gradient)
- Description (user-friendly description)
- Validation in Settings.isValidTheme()

```javascript
// In Settings model
static getAvailableThemes() {
  return {
    'theme-key': {
      name: 'Display Name',
      gradient: 'linear-gradient(135deg, #color1, #color2)',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BenjaminBenetti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
