---
trigger: always_on
description: This document contains important information for AI agents working on the Ember Native project.
---

# Agent Knowledge Base for Ember Native

This document contains important information for AI agents working on the Ember Native project.

## Project Overview

Ember Native is a framework that enables running Ember.js applications on NativeScript, allowing developers to build native mobile apps using Ember.

## WarpDrive (Ember Data v5) Integration

### Official Documentation
- **Main Docs**: https://warp-drive.io/
- **Guides**: https://warp-drive.io/guides/
- **Migration Guide**: https://raw.githubusercontent.com/warp-drive-data/warp-drive/refs/heads/main/guides/migrating/index.md
- **Requests Guide**: https://warp-drive.io/guides/the-manual/requests
- **Signals/Reactivity**: https://github.com/warp-drive-data/warp-drive/blob/main/guides/the-manual/misc/reactivity/signals.md

read https://warp-drive.io/llms-full.txt to work with warp-drive!!!

### Key Concepts

#### Modern WarpDrive (v5) - No Legacy Support
```typescript
// Store Configuration
import { useRecommendedStore } from '@warp-drive/core';
import { JSONAPICache } from '@warp-drive/json-api';

export default class StoreService extends useRecommendedStore({
  cache: JSONAPICache,
  schemas: [UserSchema],
}) {}
```

#### Schema Definition
```typescript
import { Type } from '@warp-drive/core-types/symbols';

export interface User {
  [Type]: 'user';
  id: string | null;
  name: string;
  email: string;
}

export const UserSchema = {
  type: 'user',
  identity: { name: 'id', kind: '@id' },  // Required!
  fields: [
    { name: 'name', kind: 'attribute', type: null },
    { name: 'email', kind: 'attribute', type: null },
  ],
} as const;
```

**Important**: The `identity` field is required in WarpDrive v5 schemas.

#### Using store.request() (Recommended)
```typescript
// Instead of fetch() + store.push()
const { content } = await this.store.request<{ data: User[] }>({
  url: 'https://api.example.com/users',
  method: 'GET',
  cacheOptions: {
    reload: false,           // Use cache if available
    backgroundReload: true,  // Refresh in background
    types: ['user']
  }
});

// Data is automatically cached
this.users = content.data;
```

### Signal Hooks Configuration (Ember Octane)

WarpDrive requires signal hooks for reactivity. Configure in `app/configure-signals.ts`:

```typescript
import { tagForProperty } from '@ember/-internals/metal';
import { _backburner } from '@ember/runloop';
import { consumeTag, createCache, dirtyTag, getValue } from '@glimmer/validator';
import { setupSignals } from '@warp-drive/core/configure';
import type { SignalHooks } from '@warp-drive/core/configure';

type Tag = ReturnType<typeof tagForProperty>;
const emberDirtyTag = dirtyTag as unknown as (tag: Tag) => void;

export function buildSignalConfig(): SignalHooks {
  return {
    createSignal(obj: object, key: string | symbol): Tag {
      return tagForProperty(obj, key);
    },
    
    consumeSignal(signal: Tag) {
      consumeTag(signal);
    },
    
    notifySignal(signal: Tag) {
      emberDirtyTag(signal);
    },
    
    createMemo: <F>(object: object, key: string | symbol, fn: () => F): (() => F) => {
      const memo = createCache(fn);
      return () => getValue(memo);
    },
    
    willSyncFlushWatchers: () => {
      return !!_backburner.currentInstance && _backburner._autorun !== true;
    }
  } satisfies SignalHooks;
}

setupSignals(buildSignalConfig);
```

Import in `app/app.js`:
```javascript
import './configure-signals';
```

## NativeScript Environment Polyfills

NativeScript lacks many Web APIs. Add these polyfills in `ember-native/src/setup.ts`:

### Required Polyfills
1. **queueMicrotask** - Microtask scheduling
2. **EventTarget** - Event handling
3. **AbortController/AbortSignal** - Request cancellation
4. **ReadableStream** - Stream reading
5. **WritableStream** - Stream writing
6. **TransformStream** - Stream transformation

Example implementation:
```typescript
// queueMicrotask
if (typeof globalThis.queueMicrotask === 'undefined') {
  globalThis.queueMicrotask = (callback: () => void) => {
    Promise.resolve().then(callback).catch(err => {
      setTimeout(() => { throw err; }, 0);
    });
  };
}

// EventTarget
if (typeof globalThis.EventTarget === 'undefined') {
  class EventTarget {
    private _listeners: Map<string, Set<Function>> = new Map();
    
    addEventListener(type: string, listener: Function) {
      if (!this._listeners.has(type)) {
        this._listeners.set(type, new Set());
      }
      this._listeners.get(type)!.add(listener);
    }
    
    removeEventListener(type: string, listener: Function) {
      const listeners = this._listeners.get(type);
      if (listeners) {
        listeners.delete(listener);
      }
    }
    
    dispatchEvent(event: any) {
      const listeners = this._listeners.get(event.type);
      if (listeners) {
        listeners.forEach(listener => listener(event));
      }
      return true;
    }
  }
  
  globalThis.EventTarget = EventTarget as any;
}
```

## Common Issues & Solutions

### CSS Import Errors
**Problem**: `Failed to find '~@nativescript/theme/css/core.css'`

**Solution**: 
1. Remove `~` prefix from imports in SCSS files
2. Create `postcss.config.js` with custom resolver:
```javascript
module.exports = {
  plugins: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ember-native/ember-native](https://github.com/ember-native/ember-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
