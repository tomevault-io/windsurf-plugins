---
trigger: always_on
description: **Ahtapot** is a Chrome extension for fast and secure IOC (Indicator of Compromise) analysis with multiple threat intelligence providers. It provides cybersecurity analysts with a convenient way to analyze security indicators directly from their browser.
---

# CLAUDE.md - Ahtapot IOC Analysis Extension

## Project Overview

**Ahtapot** is a Chrome extension for fast and secure IOC (Indicator of Compromise) analysis with multiple threat intelligence providers. It provides cybersecurity analysts with a convenient way to analyze security indicators directly from their browser.

### Key Information
- **Version**: 2.4.0
- **Tech Stack**: React 18 + TypeScript + Vite 5
- **Extension Type**: Chrome Manifest V3
- **License**: MIT
- **Primary Language**: TypeScript with English/Turkish i18n support

### Project Purpose
Ahtapot enables security analysts to:
1. Select any text on a webpage containing IOCs
2. Automatically detect various IOC types (IPs, domains, hashes, URLs, etc.)
3. Query multiple threat intelligence providers simultaneously
4. View consolidated analysis results in a side panel

---

## Architecture Overview

### High-Level Structure

```
ahtapot/
├── src/
│   ├── background/          # Service worker (API orchestration)
│   ├── content/            # Content scripts (IOC detection on pages)
│   ├── pages/              # Extension UI pages
│   │   ├── popup/          # Extension popup menu
│   │   ├── sidepanel/      # Main results display panel
│   │   └── options/        # Settings and API key management
│   ├── components/         # React components
│   │   └── results/        # Provider-specific result cards
│   ├── services/           # API integration layer
│   │   ├── base/           # Base service interfaces
│   │   └── tools/          # Provider implementations
│   ├── types/              # TypeScript type definitions
│   ├── utils/              # Utility functions
│   ├── i18n/               # Internationalization (EN/TR)
│   ├── config/             # Configuration files
│   └── manifest.json       # Chrome extension manifest
├── public/                 # Static assets (icons, logos)
├── dist/                   # Build output (gitignored)
└── docs/                   # Documentation
```

### Extension Architecture

Ahtapot follows Chrome Extension Manifest V3 architecture:

1. **Background Service Worker** (`src/background/service-worker.ts`)
   - Orchestrates API calls to threat intelligence providers
   - Manages API keys securely
   - Handles message passing between extension components
   - Initializes context menus

2. **Content Scripts** (`src/content/content-script.tsx`)
   - Injected into all web pages
   - Displays floating "Analyze" button on text selection
   - Detects IOCs in selected text
   - Communicates with background worker

3. **Side Panel** (`src/pages/sidepanel/`)
   - Main UI for displaying analysis results
   - Tab-based interface for each provider
   - Shows IOC detection results and threat assessments

4. **Options Page** (`src/pages/options/`)
   - API key management (add/test/save/remove)
   - General settings (language, cache retention)
   - Live API key validation

5. **Popup** (`src/pages/popup/`)
   - Quick access menu
   - Links to settings and main functionality

---

## Core Concepts

### 1. IOC Detection

**File**: `src/utils/ioc-detector.ts`

The extension automatically detects these IOC types:

| IOC Type | Examples | Pattern |
|----------|----------|---------|
| IPv4 | `192.168.1.1` | Validates 0-255 range |
| IPv6 | `2001:0db8:85a3::8a2e:0370:7334` | Full/compressed notation |
| Domain | `example.com` | Valid TLD required |
| URL | `https://example.com/path` | HTTP/HTTPS |
| MD5 | `d41d8cd98f00b204e9800998ecf8427e` | 32 hex chars |
| SHA1 | `da39a3ee5e6b4b0d3255bfef95601890afd80709` | 40 hex chars |
| SHA256 | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` | 64 hex chars |
| Email | `user@example.com` | RFC 5322 compliant |
| CVE | `CVE-2021-44228` | CVE-YYYY-NNNNN |
| Bitcoin | `1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa` | Base58/Bech32 |
| Ethereum | `0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb` | 0x + 40 hex |

**Detection Algorithm**:
- O(n) complexity with position-based deduplication
- Priority-based pattern matching (more specific patterns first)
- Prevents duplicate detection (e.g., URL containing a domain)
- Smart overlap detection using sorted ranges

### 2. Service Architecture

**Pattern**: Strategy Pattern + Service Registry

**Base Interface**: `src/services/base/BaseToolService.ts`

All provider services implement `IToolService`:
```typescript
interface IToolService {
  name: string;
  supportedIOCTypes: IOCType[];
  isConfigured(): boolean;
  supports(iocType: IOCType): boolean;
  analyze(ioc: DetectedIOC): Promise<IOCAnalysisResult>;
  analyzeBatch?(iocs: DetectedIOC[]): Promise<IOCAnalysisResult[]>;
}
```

**Service Registry**: `src/services/ServiceRegistry.ts`
- Centralized management of all provider services
- Lazy initialization (services created only when needed)
- Automatic API key injection
- ARIN service always available (no API key required)

### 3. Supported Providers

**Current Providers** (as of v2.4.0):

| Provider | Purpose | Rate Limit | Requires API Key |
|----------|---------|------------|------------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abdullahcicekli/ahtapot](https://github.com/abdullahcicekli/ahtapot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
