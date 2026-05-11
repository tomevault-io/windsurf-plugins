---
trigger: always_on
description: enableState();
---

# Invokers Library Guide for AI Agents

This document provides comprehensive information for AI agents working with the modularized Invokers library. It covers the new architecture, usage patterns, testing setup, and development considerations after the v1.5 modularization.

## 🏗️ New Modular Architecture

**Invokers** is now a hyper-modular, four-tier architecture that allows developers to import exactly what they need, from a minimal 25.8 kB core to a full-featured application framework.

### Core Philosophy
- **Standards-First**: Built on emerging W3C/WHATWG proposals
- **Modular by Design**: Import only what you need
- **Progressive Enhancement**: Start minimal, add features incrementally
- **Future-Proof**: Aligned with web platform evolution

## 🎯 Four-Tier Architecture

### **Tier 0: Core Polyfill** (`invokers`) - 25.8 kB
The foundational layer providing standards compliance.

**Contents:**
- `polyfill.ts` - CommandEvent and attribute polyfills
- `InvokerManager` - Command execution engine (empty by design)
- Core utilities and types

**Usage:**
```javascript
import 'invokers';
// Standards-compliant command/commandfor now work
```

### **Tier 1: Essential Commands**
The first commands most developers will add.

#### Base Commands (`invokers/commands/base`) - 29.2 kB
```javascript
import { registerBaseCommands } from 'invokers/commands/base';
registerBaseCommands(invokerManager);
```
**Commands**: `--toggle`, `--show`, `--hide`, `--class:*`, `--attr:*`

#### Form Commands (`invokers/commands/form`) - 30.5 kB  
```javascript
import { registerFormCommands } from 'invokers/commands/form';
registerFormCommands(invokerManager);
```
**Commands**: `--text:*`, `--value:*`, `--focus`, `--disabled:*`, `--form:*`, `--input:step`, `--text:copy`

### **Tier 2: Specialized Command Packs**

#### DOM Manipulation (`invokers/commands/dom`) - 47.1 kB
```javascript
import { registerDomCommands } from 'invokers/commands/dom';
registerDomCommands(invokerManager);
```
**Commands**: `--dom:*`, `--template:*`, data context management

#### Fetch Commands (`invokers/commands/fetch`) - ~15 kB
```javascript
import { registerFetchCommands } from 'invokers/commands/fetch';
registerFetchCommands(invokerManager);
```
**Commands**: `--fetch:get`, `--fetch:put`, `--fetch:patch`

#### WebSocket Commands (`invokers/commands/websocket`) - ~12 kB
```javascript
import { registerWebSocketCommands } from 'invokers/commands/websocket';
registerWebSocketCommands(invokerManager);
```
**Commands**: `--websocket:connect`, `--websocket:disconnect`, `--websocket:send`, `--websocket:status`, `--websocket:on:message`

#### Server-Sent Events (`invokers/commands/sse`) - ~10 kB
```javascript
import { registerSSECommands } from 'invokers/commands/sse';
registerSSECommands(invokerManager);
```
**Commands**: `--sse:connect`, `--sse:disconnect`, `--sse:status`, `--sse:on:message`, `--sse:on:event`

#### Navigation & Flow Control (`invokers/commands/navigation`) - ~8 kB
```javascript
import { registerNavigationCommands } from 'invokers/commands/navigation';
registerNavigationCommands(invokerManager);
```
**Commands**: `--navigate:to`, `--emit`, `--bind`, `--command:trigger`, `--command:delay`

#### Media & Animation (`invokers/commands/media`) - 27.7 kB
```javascript
import { registerMediaCommands } from 'invokers/commands/media';
registerMediaCommands(invokerManager);
```
**Commands**: `--media:*`, `--carousel:*`, `--scroll:*`, `--clipboard:*`

#### Browser APIs (`invokers/commands/browser`) - 25.3 kB
```javascript
import { registerBrowserCommands } from 'invokers/commands/browser';
registerBrowserCommands(invokerManager);
```
**Commands**: `--cookie:*`

#### Data Management (`invokers/commands/data`) - 45.2 kB
```javascript
import { registerDataCommands } from 'invokers/commands/data';
registerDataCommands(invokerManager);
```
**Commands**: `--data:*`, array operations, reactive data binding

#### Device APIs (`invokers/commands/device`) - 28.1 kB
```javascript
import { registerDeviceCommands } from 'invokers/commands/device';
registerDeviceCommands(invokerManager);
```
**Commands**: `--device:vibrate`, `--device:share`, `--device:geolocation:get`, `--device:battery:get`, `--device:clipboard:*`, `--device:wake-lock*`

#### Accessibility Helpers (`invokers/commands/accessibility`) - 26.8 kB
```javascript
import { registerAccessibilityCommands } from 'invokers/commands/accessibility';
registerAccessibilityCommands(invokerManager);
```
**Commands**: `--a11y:announce`, `--a11y:focus`, `--a11y:skip-to`, `--a11y:focus-trap`, `--a11y:aria:*`, `--a11y:heading-level`

### **Tier 3: Advanced Reactive Engine**

#### Event Triggers (`invokers/advanced/events`) - 42.3 kB
```javascript
import { enableEventTriggers } from 'invokers/advanced/events';
enableEventTriggers();
```
**Features**: `command-on` attribute for any DOM event

#### Expression Engine (`invokers/advanced/expressions`) - 26.2 kB
```javascript
import { enableExpressionEngine } from 'invokers/advanced/expressions';
enableExpressionEngine();
```
**Features**: `{{expression}}` interpolation in commands

#### Complete Advanced (`invokers/advanced`) - 42.4 kB
```javascript
import { enableAdvancedEvents } from 'invokers/advanced';
enableAdvancedEvents();
```
**Features**: Both event triggers and expression engine


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doeixd/invokers](https://github.com/doeixd/invokers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
