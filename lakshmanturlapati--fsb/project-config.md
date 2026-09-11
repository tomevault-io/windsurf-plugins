---
trigger: always_on
description: **Analysis Date:** 2026-02-03
---

# Coding Conventions

**Analysis Date:** 2026-02-03

## Naming Patterns

**Files:**
- Lowercase with hyphens for multi-word files: `ai-integration.js`, `universal-provider.js`, `secure-config.js`
- Single word files use lowercase: `background.js`, `content.js`, `config.js`, `popup.js`
- HTML/CSS files match their JS counterparts: `popup.html`, `popup.css`, `sidepanel.html`, `sidepanel.css`

**Functions:**
- camelCase for all functions: `handleSendMessage`, `checkContentScriptHealth`, `waitForPageReady`
- Async functions prefixed with action verbs: `async processQueue()`, `async loadFromStorage()`, `async getSecureValue()`
- Helper functions often start with `is`, `has`, `can`, `get`, `set`: `isRestrictedURL()`, `hasElementChanged()`, `getPageTypeDescription()`
- Event handlers prefixed with `handle` or action verb: `handleSendMessage()`, `stopAutomation()`

**Variables:**
- camelCase for local and instance variables: `currentSessionId`, `isRunning`, `lastError`
- SCREAMING_SNAKE_CASE for constants: `FAILURE_TYPES`, `RETRY_STRATEGIES`, `TOOL_DOCUMENTATION`, `KEY_MAPPINGS`
- Descriptive names for Maps: `activeSessions`, `contentScriptPorts`, `parameterCache`

**Classes:**
- PascalCase: `AIIntegration`, `DOMStateManager`, `PageLoadWatcher`, `UniversalProvider`, `FSBAnalytics`
- Suffix with purpose when helpful: `AutomationLogger`, `KeyboardEmulator`, `SecureConfig`

**DOM Elements:**
- camelCase matching element IDs: `chatInput`, `sendBtn`, `statusDot`
- Grouped in objects when caching: `elements.modelProvider`, `elements.saveBtn`

## Code Style

**Formatting:**
- No automated formatter configured (no Prettier/ESLint config files detected)
- Manual formatting with 2-space indentation observed throughout
- Opening braces on same line as statement
- Consistent spacing around operators

**Semicolons:**
- Always use semicolons at end of statements
- Exception: class method definitions don't need trailing semicolons

**Quotes:**
- Single quotes for strings throughout: `'running'`, `'error'`
- Template literals for string interpolation: `` `Iteration ${iterationCount}` ``

**Line Length:**
- No strict limit, but generally keep under 120 characters
- Long object definitions may span multiple lines

## Import Organization

**Order:**
- Chrome Extension pattern using `importScripts()` in service worker:
```javascript
// In background.js - load order matters for dependencies
importScripts('config.js');
importScripts('init-config.js');
importScripts('ai-integration.js');
importScripts('automation-logger.js');
importScripts('analytics.js');
importScripts('keyboard-emulator.js');
```

**Content Scripts:**
- Defined in `manifest.json` in dependency order:
```json
"js": ["automation-logger.js", "content.js"]
```

**Cross-Environment Exports:**
- Use conditional exports for modules that work in multiple contexts:
```javascript
// Export for service workers
if (typeof self !== 'undefined') {
  self.config = config;
}

// Export for content scripts
if (typeof window !== 'undefined') {
  window.BrowserAgentConfig = config;
}

// CommonJS fallback
if (typeof module !== 'undefined' && module.exports) {
  module.exports = { function1, function2 };
}
```

## Error Handling

**Patterns:**
- Use try-catch for async operations and API calls
- Return structured objects with `success` boolean:
```javascript
return { success: true, waitTime, method: 'event-driven' };
return { success: false, error: error.message };
```

- Log errors before throwing or returning:
```javascript
} catch (error) {
  automationLogger.error('AI request failed', { error: error.message });
  return this.createFallbackResponse(task, error);
}
```

**Error Classification:**
- Define error types as constants (see `background.js` line 528-536):
```javascript
const FAILURE_TYPES = {
  COMMUNICATION: 'communication',
  DOM: 'dom',
  SELECTOR: 'selector',
  NETWORK: 'network',
  TIMEOUT: 'timeout',
  PERMISSION: 'permission',
  BF_CACHE: 'bfcache'
};
```

**Recovery Strategies:**
- Map error types to recovery handlers:
```javascript
const RETRY_STRATEGIES = {
  [FAILURE_TYPES.COMMUNICATION]: 'reconnect_retry',
  [FAILURE_TYPES.DOM]: 'wait_retry',
  // ...
};
```

**Retry Logic:**
- Use exponential backoff for retries:
```javascript
const delay = baseDelay * Math.pow(2, attempt);
await new Promise(resolve => setTimeout(resolve, delay));
```

## Logging

**Framework:** Custom `AutomationLogger` class (`automation-logger.js`)

**Log Levels:**
- `error`: Critical failures
- `warn`: Recoverable issues, deprecation warnings
- `info`: Important state changes, session events
- `debug`: Detailed operation tracing

**Patterns:**
```javascript
// Standard logging
automationLogger.info('Automation session started', { sessionId, task, tabId });
automationLogger.error('AI request failed', { error: error.message });
automationLogger.debug('Content script ready via port', { tabId });

// Specialized logging methods for different concerns:
automationLogger.logSessionStart(sessionId, task, tabId);
automationLogger.logIteration(sessionId, iterationCount, domHash, stuckCounter);
automationLogger.logAction(sessionId, action, result);
automationLogger.logAIResponse(sessionId, reasoning, actions, taskComplete);
automationLogger.logStuckDetection(sessionId, stuckCounter, lastActions);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LakshmanTurlapati/FSB](https://github.com/LakshmanTurlapati/FSB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
