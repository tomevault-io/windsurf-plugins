---
trigger: always_on
description: **Focusan** is a Chrome browser extension designed to help users maintain focus and attention by blocking distracting websites with motivational reminders. The extension uses Chrome's Manifest V3 architecture and provides flexible scheduling, statistics tracking, and focus session management.
---

# Focusan - AI Assistant Documentation

## Application Overview

**Focusan** is a Chrome browser extension designed to help users maintain focus and attention by blocking distracting websites with motivational reminders. The extension uses Chrome's Manifest V3 architecture and provides flexible scheduling, statistics tracking, and focus session management.

**Tech Stack**: TypeScript, React, Chrome Extension Manifest V3, Vite, Zod

## Project Structure

### Source Code (`src/`)

#### Background Service (`src/background/`)
- **index.ts** - Main background service worker entry point
  - Coordinates all extension functionality
  - Handles tab navigation and blocking
  - Manages alarms and focus sessions
- **dnr-manager.ts** - Declarative Net Request (DNR) rules manager
  - Builds blocking rules from sites and focus sessions
  - Updates Chrome's declarativeNetRequest API
- **handlers.ts** - Message handlers for communication with UI
- **alarms.ts** - Alarm system for scheduled tasks

#### Shared Domain Logic (`src/shared/domain/`)
- **focus-sessions.ts** - Pomodoro-style focus session management
- **schedule.ts** - Time-based blocking rules
- **conditional-rules.ts** - Advanced conditional blocking logic
- **stats.ts** - Statistics tracking for blocked attempts
- **achievements.ts** - Gamification system
- **categories.ts** - Site categorization system

#### Storage Layer (`src/shared/storage/`)
- **storage.ts** - Unified storage abstraction with type-safe operations
- **schemas.ts** - Zod schemas for runtime validation
- **migrations.ts** - Data migration system for version updates

#### Utilities (`src/shared/utils/`)
- **domain.ts** - Domain normalization utilities
  - `normalizeHost()` - Converts URLs to normalized hostname format
  - `hostToRegex()` - Generates regex patterns for DNR rules

#### Messaging System (`src/shared/messaging/`)
- **contracts.ts** - Type-safe message contracts
- **client.ts** - Client for sending messages to background

#### Internationalization (`src/shared/i18n/`)
- **translations.ts** - Translation strings
- **index.ts** - i18n system for UI text

#### UI Components
- **src/popup/** - Extension popup (React)
  - Quick site blocking/unblocking
  - Focus session controls
  - Site counter display

- **src/options/** - Settings page (React)
  - Blocked sites management
  - Schedule configuration
  - Categories and conditional rules
  - Statistics dashboard

- **src/pages/blocked/** - Block notification page (React)
  - Motivational messages
  - Temporary allowance options

- **src/pages/diagnostics/** - Debug/troubleshooting interface (React)

#### Configuration
- **public/manifest.json** - Chrome extension manifest (source)
- **vite.config.ts** - Vite build configuration
- **tsconfig.json** - TypeScript configuration
- **styles.css** - Global styles

#### Assets
- **icons/** - Extension icons (16x16, 32x32, 48x48, 128x128)

## Key Architecture Patterns

### Modern TypeScript Stack
- **TypeScript** - Full type safety across the codebase
- **Zod** - Runtime validation for storage data
- **React** - Modern UI components
- **Vite** - Fast build system with HMR
- **ES Modules** - Service worker uses modern module syntax

### Data Format
Sites are stored as structured objects:
- **Format**: `[{ host: "example.com", addedAt: 1234567890, category: null, schedule: null }]`

The storage.ts module provides type-safe access to all site data.

### Blocking Mechanism
Uses Chrome's declarativeNetRequest API (Manifest V3 requirement):
1. Service worker builds rules from blocked sites list
2. Rules are registered with `chrome.declarativeNetRequest.updateDynamicRules()`
3. Chrome blocks matching requests at network level
4. Blocked navigation triggers redirect to blocked.html

### Focus Sessions
Pomodoro-style work sessions that temporarily allow specific sites:
- `activeFocusSessionSites` - Set of allowed hosts during active session
- `focusSessionEndTime` - Session expiration timestamp
- Managed via alarms API for accurate timing

## Chrome Extension Permissions

Required permissions in manifest.json:
- `storage` - For saving blocked sites and settings (chrome.storage.sync)
- `tabs` - For detecting current tab and site blocking status
- `declarativeNetRequest` - For network-level site blocking (Manifest V3)
- `notifications` - For user notifications about blocks/focus sessions
- `webNavigation` - For detecting navigation to blocked sites
- `alarms` - For scheduling time-based features
- `http://*/*`, `https://*/*` - Host permissions for blocking all HTTP(S) sites

## Internationalization (i18n)

Supports multiple languages:
- Translation keys in translations.js
- Dynamic language detection from browser settings
- Manual language override via storage
- Used in all UI components (popup, options, blocked page)

Current languages: Russian (primary), English

## Development Guidelines

### Adding New Blocked Sites
Sites are normalized before storage:
1. Parse URL to extract hostname
2. Remove `www.` prefix
3. Convert to lowercase

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nomad06/focusan](https://github.com/Nomad06/focusan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
