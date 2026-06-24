---
trigger: always_on
description: This file provides specialized guidance for different agent types when working on the Yasumaro Chrome extension project.
---

# AGENTS.md

This file provides specialized guidance for different agent types when working on the Yasumaro Chrome extension project.

> **Note:** For general contribution guidelines (setup, testing, PR workflow), see [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Overview

This is a **Manifest V3 Chrome extension** with a modular architecture:
- Service worker background script coordinates all operations
- Content script tracks user engagement on web pages
- Popup UI provides configuration and testing interface
- Modular client classes handle AI providers and Obsidian integration

### Quick References

| For Documentation | See |
|------------------|-----|
| Project Architecture | [dev-docs/DESIGN_SPECIFICATIONS.md](dev-docs/DESIGN_SPECIFICATIONS.md) |
| Architecture Decisions | [dev-docs/ADR/](dev-docs/ADR/) |
| Error Codes | [dev-docs/ERROR_CODES.md](dev-docs/ERROR_CODES.md) |
| Contribution Guide | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Accessibility Guide | [docs/ACCESSIBILITY.md](docs/ACCESSIBILITY.md) |
| i18n Guide | [docs/i18n-guide.md](docs/i18n-guide.md) |

---

## Quick Start

```bash
npm install              # Install dependencies
npm run build:watch      # Build and watch for development changes
npm validate             # Type check + run tests (pre-commit gate)
```

### Loading the Extension

1. Run `npm build` to build the extension
2. Open Chrome and navigate to `chrome://extensions`
3. Enable "Developer mode" (toggle in top-right)
4. Click "Load unpacked" and select the `dist/chromium-mv3` directory
5. The extension is now installed

---

## For Feature Development Agents

### Architecture Context

The extension follows a modular design pattern:

```
Service Worker (src/background/)
  ├── ObsidianClient → Obsidian Local REST API
  ├── AIClient (multiple implementations) → AI Providers
  ├── localAiClient → Local AI provider (Ollama, etc.)
  ├── sessionAlarmsManager → Session timeout management
  ├── Mutex / ServiceWorkerContext → Concurrency management
  └── recordingLogic → Core recording orchestration

Popup UI (src/popup/)
  ├── navigation.ts → Tab management
  ├── domainFilter.ts → Domain filter settings
  ├── main.ts → Core popup logic
  ├── ublockImport/ → uBlock filter import functionality
  ├── settings/ → Settings management
  └── utils/ → Shared utilities (focusTrap, i18n, etc.)

Dashboard (src/dashboard/)
  ├── dashboard.html → Settings configuration interface
  └── dashboard.ts → Dashboard logic

Offscreen (src/offscreen/)
  └── offscreen.ts → DOM operations requiring offscreen document

Content Scripts (src/content/)
  ├── loader.ts → Injection orchestrator
  └── extractor.ts → DOM content extraction
```

### Key Patterns to Follow

1. **Modular Design**: Keep specific functionality in dedicated client classes
2. **Async/Await**: All API calls should use async/await with proper error handling
3. **Chrome Extension APIs**: Use appropriate Chrome APIs (storage, tabs, scripting)
4. **Message Passing**: Communicate between components using Chrome's message passing API
5. **Error Handling**: Always implement try-catch blocks with user notifications

### Adding New Features

| Feature Type | Location | Notes |
|--------------|----------|-------|
| UI features | `src/popup/` (HTML/CSS/TS) | Follow accessibility patterns (see ACCESSIBILITY.md) |
| Dashboard settings | `src/dashboard/` (HTML/CSS/TS) | Settings management interface |
| uBlock Import | `src/popup/ublockImport/` | Filter list import functionality |
| Background processing | `src/background/` service-worker.ts | Use modular client classes |
| Local AI Integration | `src/background/localAiClient.ts` | Ollama and other local providers |
| Page interaction | `src/content/` extractor.ts | Consider CSP restrictions |
| Storage | `src/utils/storage.ts` | Use StorageKeys constant |
| API Key Encryption | `src/utils/crypto.ts` | PBKDF2 + AES-GCM encryption |
| PII Masking | `src/utils/piiSanitizer.ts` | Privacy-preserving data handling |
| DOM operations | `src/offscreen/` offscreen.ts | For operations requiring offscreen document |
| Trust Database | `src/utils/trustDb/` | Domain trust verification with 3-step check |
| Permission Manager | `src/utils/permissionManager.ts` | chrome.permissions API wrapper + denied domain tracking |
| CSP Settings | `src/dashboard/cspSettings.ts` | Conditional CSP configuration for AI providers |

**Before implementing major features**, review [dev-docs/ADR/](dev-docs/ADR/) for existing architectural decisions and consistency.

### Critical Considerations

- **i18n**: All user-facing text must use data-i18n attributes (see [i18n-guide.md](docs/i18n-guide.md))
- **Accessibility**: Follow WCAG 2.1 Level AA guidelines (see [docs/ACCESSIBILITY.md](docs/ACCESSIBILITY.md))
- **Manifest V3**: No background scripts, use service workers
- **CSP**: Adhere to Content Security Policy
- **Offscreen API**: Use offscreen documents for DOM operations that cannot run in service workers

---

## For Code Review Agents

### Security Checklist

- [ ] No hardcoded API keys or sensitive data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [armaniacs/yasumaro](https://github.com/armaniacs/yasumaro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
