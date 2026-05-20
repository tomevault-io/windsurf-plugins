---
trigger: always_on
description: TabBoost is a Chrome extension that enhances browser tab efficiency with features inspired by Arc Browser. The extension provides link preview, split-screen browsing, tab management, and other productivity features.
---

# Copilot Instructions for TabBoost

## Project Overview

TabBoost is a Chrome extension that enhances browser tab efficiency with features inspired by Arc Browser. The extension provides link preview, split-screen browsing, tab management, and other productivity features.

**Key Technologies:**
- Chrome Extension Manifest V3
- Vanilla JavaScript (ES2021+)
- Webpack for bundling
- Jest for testing
- ESLint for code quality

## Architecture

### Directory Structure
- `src/js/` - Core JavaScript files
  - `background.js` - Service worker (background script)
  - `contentScript.js` - Content script injected into web pages
  - `splitView/` - Split view functionality
- `src/popup/` - Extension popup UI
- `src/options/` - Extension options page
- `src/styles/` - CSS stylesheets
- `src/assets/` - Icons and other assets
- `src/_locales/` - Internationalization (i18n) files
- `tests/` - Jest test files
- `rules/` - Declarative Net Request rules

### Key Components
1. **Background Service Worker** (`background.js`) - Handles extension lifecycle, commands, and tab management
2. **Content Script** (`contentScript.js`) - Injected into web pages for link preview and split view
3. **Split View** (`splitView/`) - Side-by-side page viewing functionality
4. **Popup UI** - Quick access to extension features
5. **Options Page** - User preferences and settings

## Coding Standards

### JavaScript
- Use ES2021+ features
- Follow ESLint configuration in `.eslintrc.json`
- Indentation: 2 spaces
- Quotes: Double quotes
- Semicolons: Required
- Line endings: Unix (LF)

### Code Style
```javascript
// Use descriptive variable names and destructure returned objects
const { isValid } = validateUrl(url);

// Use async/await for asynchronous operations
async function fetchData() {
  const result = await chrome.storage.sync.get("key");
  return result;
}

// Use Chrome Extension APIs correctly
chrome.tabs.query({ active: true }, (tabs) => {
  // Handle tabs
});
```

### Chrome Extension Best Practices
1. **Manifest V3** - Use service workers instead of background pages
2. **Content Security Policy** - Follow strict CSP defined in manifest.json
3. **Permissions** - Request minimal permissions needed
4. **Message Passing** - Use `chrome.runtime.sendMessage` for communication
5. **Storage** - Use `chrome.storage.sync` for user preferences

## Build and Test Commands

**Note:** This project supports both `npm` and `yarn` package managers. The examples below use `npm`, but you can substitute with `yarn` commands.

### Development
```bash
npm run dev          # Build in development mode
npm run start        # Start webpack dev server
npm run build        # Build for production
npm run clean        # Clean build artifacts
```

### Testing
```bash
npm test             # Run all tests
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Generate coverage report
npm run test:ci      # Run tests in CI mode
```

### Code Quality
```bash
npm run format       # Format code with Prettier
npm run format:check # Check code formatting
npx eslint src/      # Run ESLint manually (not in npm scripts)
```

### Release
```bash
npm run validate     # Validate build
npm run zip          # Create distribution zip
npm run release      # Build, validate, and create zip
```

## Testing Guidelines

- **Test Framework:** Jest with jsdom environment
- **Chrome API Mocking:** Use `jest-chrome` for mocking Chrome APIs
- **Coverage Targets:**
  - Statement coverage: ≥60%
  - Branch coverage: ≥50%
  - Function coverage: ≥60%
  - Line coverage: ≥60%

### Test Structure
```javascript
describe("Component or Function", () => {
  beforeEach(() => {
    // Setup mocks
    chrome.storage.sync.get.mockImplementation((keys, callback) => {
      callback({ key: "value" });
    });
  });

  test("should do something specific", () => {
    // Test implementation
  });
});
```

## Security Considerations

1. **URL Validation** - Always validate URLs before processing
2. **CSP Compliance** - Respect Content Security Policy headers
3. **XSS Prevention** - Sanitize user input and dynamic content
4. **Frame Security** - Handle X-Frame-Options correctly
5. **Permission Management** - Request minimal permissions

### Security Patterns
```javascript
// URL validation example
import { validateUrl } from "../utils/utils.js";

const result = validateUrl(url);
if (!result.isValid) {
  console.error("Invalid URL:", result.message);
  return;
}
```

## Internationalization (i18n)

The extension supports multiple languages via Chrome's i18n system:
- English (en) - default
- Chinese Simplified (zh_CN)
- Chinese Traditional (zh_TW)
- Japanese (ja)
- Korean (ko)
- French (fr)
- German (de)
- Spanish (es)
- Russian (ru)
- Thai (th)

### i18n Best Practices
1. All user-facing strings should be in `src/_locales/{locale}/messages.json`
2. Use `chrome.i18n.getMessage("key")` to retrieve localized strings
3. In manifest.json, use `__MSG_key__` format
4. Always add new strings to all locale files or at minimum to English (en)

### Example Usage
```javascript
// In JavaScript
const message = chrome.i18n.getMessage("appName");


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samzong/chrome-tabboost](https://github.com/samzong/chrome-tabboost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
