---
trigger: always_on
description: Validates:
---

# LLMFeeder - AI Agent & Contributor Guide

This guide is designed for AI agents (like Claude, ChatGPT, etc.) and human contributors working on the LLMFeeder browser extension. It provides a comprehensive reference for understanding the codebase, development workflow, and testing procedures.

## Table of Contents

1. [Quick Reference](#quick-reference)
2. [Repository Structure](#repository-structure)
3. [Development Workflow](#development-workflow)
4. [Extension Architecture](#extension-architecture)
5. [Local Testing Guide](#local-testing-guide)
6. [Debug Mode & Logging](#debug-mode--logging)
7. [Common Development Tasks](#common-development-tasks)
8. [Build & Release Process](#build--release-process)
9. [CI/CD Pipeline](#cicd-pipeline)

---

## Quick Reference

```bash
# Build extensions
./scripts/build.sh chrome    # Chrome only
./scripts/build.sh firefox   # Firefox only
./scripts/build.sh all       # All packages
make chrome                  # Alternative: using Make

# Start local test server
python3 -m http.server 8080

# Git workflow
git checkout -b feature/your-feature-name
# ... make changes ...
git add extension/
git commit -m "feat: description"
git push origin feature/your-feature-name
```

**Key Files for Feature Development:**
- `extension/content.js` - Core conversion logic (main content extraction, markdown)
- `extension/popup.js` - Popup UI and settings handling
- `extension/popup.html` - UI structure
- `extension/styles.css` - Styling
- `extension/manifest.json` - Extension configuration

**Testing Files:**
- `testbench.html` - Comprehensive test page
- `iframe-content-test.html` - Nested iframe content for testing

---

## Repository Structure

```
LLMFeeder/
├── extension/                 # Browser extension source
│   ├── icons/                # Extension icons (16, 48, 128px)
│   ├── libs/                 # Third-party libraries
│   │   ├── readability.js    # Mozilla content extraction
│   │   ├── turndown.js       # HTML to Markdown
│   │   └── browser-polyfill.js # Cross-browser compatibility
│   ├── manifest.json         # Extension manifest (MV3 base)
│   ├── popup.html            # Popup UI
│   ├── popup.js              # Popup logic (18KB)
│   ├── styles.css            # Popup styling
│   ├── content.js            # Content script (42KB) - CORE LOGIC
│   └── background.js         # Background script for keyboard shortcuts
│
├── scripts/
│   └── build.sh              # Build script (240 lines)
│
├── .github/workflows/
│   ├── pr-validation.yml     # PR build validation
│   └── release.yml           # Release asset upload
│
├── testbench.html             # Main testing page
├── iframe-content-test.html   # Nested iframe test content
├── Makefile                   # Build automation
└── README.md                  # User documentation
```

---

## Development Workflow

### Step 1: Create a Feature Branch

```bash
git checkout main
git pull origin main
git checkout -b feature/your-feature-name
```

### Step 2: Make Changes

Edit files in the `extension/` directory based on your feature:

| Change Type | Files to Modify |
|-------------|-----------------|
| New UI controls | `popup.html`, `popup.js`, `styles.css` |
| New settings | `popup.js` (add to loadSettings/saveSettings) |
| Content extraction logic | `content.js` |
| Keyboard shortcuts | `background.js`, `manifest.json` |
| Permissions | `manifest.json` |

### Step 3: Build and Test Locally

```bash
# Build Chrome extension
./scripts/build.sh chrome

# Extract for local testing
unzip -q dist/LLMFeeder-Chrome-v*.zip -d /tmp/llmfeeder-test
# OR just use the chrome-unpacked directory if created
```

### Step 4: Load in Browser

1. Open `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the extracted extension directory (or `extension/` folder directly)

### Step 5: Test with Test Bench

```bash
# In project root
python3 -m http.server 8080
```

Then navigate to `http://localhost:8080/testbench.html` and test your changes.

### Step 6: Commit and Push

```bash
git add extension/
git commit -m "feat: description of changes"
git push origin feature/your-feature-name
```

---

## Extension Architecture

### Component Communication

```
┌─────────────────┐     messages     ┌─────────────────┐
│   popup.js      │ ───────────────▶ │   content.js    │
│  (UI/Settings)  │                  │ (Core Logic)    │
└─────────────────┘                  └─────────────────┘
        ▲                                    │
        │                                    │
        │ browser storage                   │
        │                                    ▼
┌─────────────────┐              ┌─────────────────┐
│ browser.storage │              │   Turndown.js   │
│  (sync prefs)   │              │  (HTML→MD)      │
└─────────────────┘              └─────────────────┘
                                         ▲
                                         │
┌─────────────────┐                      │
│ background.js   │ ─────────────────────┘
│ (shortcuts)     │
└─────────────────┘
```

### Content Script (`content.js`) - Core Functions

| Function | Purpose |
|----------|---------|
| `convertToMarkdown(settings)` | Main entry point for conversion |
| `extractMainContent(doc)` | Uses Readability.js for article content |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jatinkrmalik/LLMFeeder](https://github.com/jatinkrmalik/LLMFeeder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
