---
trigger: always_on
description: **Comet-AI** is a cross-platform AI-powered browser with advanced automation capabilities.
---

# Comet-AI Project - Agent Tasks & Status

## Project Overview
**Comet-AI** is a cross-platform AI-powered browser with advanced automation capabilities.

### Components
| Component | Technology | Path |
|-----------|-----------|------|
| Desktop Browser | Electron | `comet-browser/` |
| Mobile App | Flutter | `flutter_browser_app/` |
| Backend Services | FastAPI | `presenton/servers/fastapi/` |

---

## ✅ COMPLETED TASKS

### LLM/AI Crawlability (2026-04-06)

**Status:** ✅ COMPLETE

**Files Created:**
- `Landing_Page/public/sitemap.xml` - XML sitemap for all doc pages
- `Landing_Page/public/llms.txt` - Plain text index for AI/LLM crawlers
- `Landing_Page/public/.well-known/ai-plugin.json` - AI plugin manifest
- `Landing_Page/src/app/llms.txt/route.ts` - Dynamic llms.txt generation
- `Landing_Page/src/app/docs/metadata.ts` - Doc-specific metadata helpers
- `Landing_Page/public/robots.txt` - Updated for AI crawlers (GPTBot, ClaudeBot, Gemini, Perplexity, etc.)

**Files Modified:**
- `Landing_Page/src/app/layout.tsx` - Enhanced JSON-LD structured data (SoftwareApplication, TechArticle, CollectionPage schemas)

**Features Implemented:**
- Full AI crawler support (ChatGPT, Claude, Gemini, Perplexity, etc.)
- llms.txt plain text documentation index
- Enhanced sitemap with all 18 doc pages
- JSON-LD structured data for semantic understanding
- robots.txt allowing all AI crawlers

---

### Docs Search Feature (2026-04-06)

**Status:** ✅ COMPLETE

**Files Created:**
- `Landing_Page/src/lib/search-index.ts` - Full search index with 40+ indexed items
- `Landing_Page/src/app/api/search/route.ts` - Search API endpoint
- `Landing_Page/src/components/docs/SearchModal.tsx` - Full-featured search modal UI

**Files Modified:**
- `Landing_Page/src/app/docs/layout.tsx` - Integrated search modal with ⌘K shortcut

**Features:**
- Fuzzy search across all documentation
- Keyboard navigation (↑↓ to navigate, Enter to select, Esc to close)
- Search results categorized by type (page, section, command, API, guide)
- Quick links when search is empty
- Debounced search with 150ms delay
- URL: /api/search?q=query

**Indexed Content:**
- All 16 documentation pages
- 30+ AI commands (NAVIGATE, SHELL_COMMAND, CREATE_PDF, OCR, etc.)
- IPC handlers (browser, shell, automation, AI, sync)
- Security features and patterns
- Plugin system and SDK

---

## 📋 ACTIVE TASKS

### 1. Background Automation Service (HIGH PRIORITY)

**Status:** ✅ IMPLEMENTED

**Description:** Create a background service that runs scheduled tasks even when browser is closed. Runs as OS-level service (SYSTEM on Windows, LaunchDaemon on macOS).

**Requirements:**
- ✅ Run when user asks AI to schedule tasks (e.g., "generate PDF at 8am")
- ✅ Run even when no user logged in (SYSTEM user)
- ✅ Default storage: `~/Documents/Comet-AI/`
- ✅ AI asks user where to save files
- ✅ Notifications to both desktop and mobile
- ✅ PDF viewing on mobile
- ✅ Model selection via popup/modal
- ✅ Both browser and service can handle tasks simultaneously

**Architecture:**
```
┌─────────────────────────────────────────────────────────────────┐
│  comet-ai-service.exe (Background Service)                      │
│  • Runs as SYSTEM user                                         │
│  • System tray icon only                                       │
│  • Executes scheduled tasks                                     │
│  • ~30-50MB RAM                                                │
└─────────────────────────────────────────────────────────────────┘
```

**Implementation Phases:**
| Phase | Task | Status |
|-------|------|--------|
| 1.1 | Service entry point (`src/service/service-main.js`) | ✅ Complete |
| 1.2 | Tray manager (`src/service/tray-manager.js`) | ✅ Complete |
| 1.3 | Windows service installer (`scripts/install-service.js`) | ✅ Complete |
| 1.4 | macOS LaunchDaemon setup (`scripts/install-service.sh`) | ✅ Complete |
| 2.1 | Scheduler with node-cron (`src/service/scheduler.js`) | ✅ Complete |
| 2.2 | Task queue with retry (`src/service/task-queue.js`) | ✅ Complete |
| 2.3 | Storage management (`src/service/storage.js`) | ✅ Complete |
| 3.1 | Model selector popup (`src/components/ai/SchedulingModal.tsx`) | ✅ Complete |
| 3.2 | Ollama integration (`src/service/ollama-manager.js`) | ✅ Complete |
| 4.1 | Desktop notifications (`src/service/notifications.js`) | ✅ Complete |
| 4.2 | Mobile push notifications (`src/service/mobile-notifier.js`) | ✅ Complete |
| 5.1 | PDF sync for mobile (`src/service/pdf-sync.js`) | ✅ Complete |
| 5.2 | Mobile PDF viewer (`lib/pages/pdf_viewer_page.dart`) | ✅ Complete |
| 6.1 | Mobile automation page (`lib/pages/automation_page.dart`) | ✅ Complete |
| 6.2 | Remote settings page (`lib/pages/remote_settings_page.dart`) | ✅ Complete |
| 6.3 | IPC bridge (`src/service/ipc-service.js`) | ✅ Complete |
| 6.4 | Sleep handler (`src/service/sleep-handler.js`) | ✅ Complete |

**Files Created:**
```
comet-browser/
├── src/service/
│   ├── service-main.js           # ✅ Entry point (245 lines)
│   ├── tray-manager.js           # ✅ System tray (190 lines)
│   ├── scheduler.js              # ✅ Cron scheduler (280 lines)
│   ├── task-queue.js            # ✅ Priority queue (420 lines)
│   ├── storage.js               # ✅ File management (200 lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Preet3627/Comet-AI](https://github.com/Preet3627/Comet-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
