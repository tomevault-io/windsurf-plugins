---
trigger: always_on
description: The `@snowplow/browser-tracker-core` library provides browser-specific tracking functionality built on top of tracker-core. It handles DOM interactions, browser storage, cookies, and browser-specific event tracking features.
---

# Browser Tracker Core Library - CLAUDE.md

## Module Overview

The `@snowplow/browser-tracker-core` library provides browser-specific tracking functionality built on top of tracker-core. It handles DOM interactions, browser storage, cookies, and browser-specific event tracking features.

## Core Responsibilities

- **Browser Detection**: User agent, viewport, screen detection
- **Storage Management**: Cookies, localStorage, sessionStorage
- **DOM Helpers**: Cross-domain linking, form tracking utilities
- **Session Management**: Session tracking and storage
- **Page View Tracking**: Page view IDs and activity tracking
- **Browser Plugins**: Plugin system for browser features

## Architecture Patterns

### Browser Plugin Pattern
```typescript
// ✅ Correct: Typed plugin interface
export interface BrowserPlugin {
  activateBrowserPlugin?: (tracker: BrowserTracker) => void;
  contexts?: () => Array<SelfDescribingJson>;
  logger?: (logger: Logger) => void;
  beforeTrack?: (payloadBuilder: PayloadBuilder) => void;
  afterTrack?: (payload: Payload) => void;
}

// ❌ Wrong: Untyped plugin
const plugin = { activate: (t: any) => {} };
```

### Tracker State Management
```typescript
// ✅ Correct: Shared state pattern
export interface SharedState {
  bufferFlushers: Array<(sync?: boolean) => void>;
  hasLoaded: boolean;
  registeredOnLoadHandlers: Array<Function>;
  contextProviders: Array<ContextProvider>;
}

// ❌ Wrong: Global variables
let trackerState = {}; // Avoid global state
```

### Storage Abstraction Pattern
```typescript
// ✅ Correct: Storage interface
export interface Storage {
  getItem(key: string): string | null;
  setItem(key: string, value: string): void;
  removeItem(key: string): void;
}

// ❌ Wrong: Direct localStorage access
localStorage.setItem('key', 'value'); // Use abstraction
```

## Critical Import Patterns

### Browser-Specific Exports
```typescript
// index.ts - Browser features
export * from './tracker/types';
export * from './helpers';
export * from './detectors';
export * from './proxies';
export * from './plugins';
export * from './state';
export { Tracker } from './tracker';
```

### Helper Utilities
```typescript
// ✅ Correct: Import from helpers
import { getCookieValue, setCookie } from './helpers';

// ❌ Wrong: Reimplementing browser utils
function getCookie(name: string) { /* custom */ }
```

## Cookie Management

### Cookie Storage Pattern
```typescript
// ✅ Correct: Use cookie storage abstraction
export class CookieStorage implements Storage {
  constructor(
    private cookieName: string,
    private cookieDomain?: string,
    private cookieLifetime?: number
  ) {}
  
  setItem(key: string, value: string): void {
    setCookie(this.cookieName, value, this.cookieLifetime);
  }
}

// ❌ Wrong: Direct document.cookie manipulation
document.cookie = `name=value; domain=.example.com`;
```

### ID Cookie Pattern
```typescript
// Standard ID cookie structure
interface IdCookie {
  userId: string;          // Domain user ID
  sessionId: string;       // Session ID
  sessionIndex: number;    // Session count
  eventIndex: number;      // Event count in session
  lastActivity: number;    // Last activity timestamp
}
```

## Session Management

### Session Storage Pattern
```typescript
// ✅ Correct: Session data structure
export interface SessionData {
  sessionId: string;
  previousSessionId?: string;
  sessionIndex: number;
  userId: string;
  firstEventId?: string;
  firstEventTimestamp?: number;
}

// ❌ Wrong: Unstructured session data
const session = { id: '123', data: {} };
```

### Activity Tracking
```typescript
// Activity callback configuration
export interface ActivityTrackingConfiguration {
  minimumVisitLength: number;
  heartbeatDelay: number;
  callback?: ActivityCallback;
}
```

## DOM Interaction Patterns

### Cross-Domain Linking
```typescript
// ✅ Correct: Decorator pattern
export function decorateQuerystring(
  url: string,
  name: string,
  value: string
): string {
  const [urlPath, anchor] = url.split('#');
  const [basePath, queries] = urlPath.split('?');
  // ... decoration logic
  return decorated;
}

// ❌ Wrong: String concatenation
url += '?_sp=' + value; // Fragile
```

### Form Element Helpers
```typescript
// Get form element values safely
export function getElementValue(element: HTMLElement): string | null {
  if (element instanceof HTMLInputElement) {
    return element.value;
  }
  if (element instanceof HTMLTextAreaElement) {
    return element.value;
  }
  return null;
}
```

## Browser Detection

### Detector Functions
```typescript
// ✅ Correct: Feature detection
export function hasLocalStorage(): boolean {
  try {
    const test = 'localStorage';
    localStorage.setItem(test, test);
    localStorage.removeItem(test);
    return true;
  } catch (e) {
    return false;
  }
}

// ❌ Wrong: Assume availability
if (localStorage) { /* use it */ } // May throw
```

### Browser Properties
```typescript
// Standard browser context
export interface BrowserProperties {
  viewport: [number, number];
  documentSize: [number, number];
  resolution: [number, number];
  colorDepth: number;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowplow/snowplow-javascript-tracker](https://github.com/snowplow/snowplow-javascript-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
