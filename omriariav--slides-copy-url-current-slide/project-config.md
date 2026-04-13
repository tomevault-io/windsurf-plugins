---
trigger: always_on
description: This is a Chrome extension that adds a "Copy current slide link" button to Google Slides using **two integration methods**: Quick Actions Menu (native) and Share Dialog (overlay). Features event-driven architecture with Google-style UI elements.
---

# Cursor Rules for Google Slides Extension Project

## Project Context
This is a Chrome extension that adds a "Copy current slide link" button to Google Slides using **two integration methods**: Quick Actions Menu (native) and Share Dialog (overlay). Features event-driven architecture with Google-style UI elements.

**🚀 Published on Chrome Web Store:** https://chromewebstore.google.com/detail/google-slides-current-sli/iifbobbbmgboednjjnlegdbpgdgpldfl

## Core Principles

### 1. Dual Integration Strategy
- **Quick Actions Menu**: Native Google menu integration for seamless UX
- **Share Dialog Overlay**: Overlay positioning for broader compatibility
- **Respect Google's UI**: Never manually manipulate Google's native menu behavior
- **Progressive Enhancement**: Core functionality works, enhancements add value
- **Proactive Injection**: Inject immediately on page load for instant availability

### 2. Architecture Patterns
- **Always prefer event-driven architecture over periodic scanning**
- **Use native integration when possible, overlay as fallback**
- **Implement one-time injection with smart state management**
- **Apply defensive programming with graceful degradation**
- **Let Google handle what Google does best (menu state, closing behavior)**
- **Implement proactive injection for immediate user availability**

### 3. Performance Guidelines
- **Avoid setInterval for DOM monitoring - use event listeners instead**
- **Implement one-time injection patterns to prevent duplicate operations**
- **Use production-aware logging to minimize console noise**
- **Prefer targeted DOM queries over broad searches**
- **Cache slide ID updates only when necessary**
- **Use proactive injection to eliminate first-use delays**

### 4. State Management
- **Use a centralized state object for all extension state**
- **Implement injection flags to prevent duplicate injections**
- **Track fallback checks to ensure one-time-only operations**
- **Keep state minimal - only store what's necessary**
- **Always reset state flags in appropriate cleanup functions**
- **Use throttled logging for performance optimization**

## Coding Standards

### Function Naming Conventions
- **Action functions:** `injectCurrentSlideOption()`, `showLinkCopiedTooltip()`
- **Query functions:** `getCurrentSlideId()`, `buildSlideUrl()`  
- **Setup functions:** `setupQuickActionsMenuDetection()`, `scanForExistingQuickActionsMenu()`
- **Handler functions:** `createQuickActionsClickHandler()`, `handleMenuClick()`
- **Utility functions:** `throttleLog()`, `updateSlideId()`
- **Initialization functions:** `initializeQuickActionsIntegration()`, `setupProactiveInjection()`

### Variable Naming
- **Use descriptive names:** `quickActionsInjected` not `injected`
- **Boolean flags:** `isInjecting`, `quickActionsInjected`, `fallbackCheckDone`
- **DOM elements:** `copyLinkButton`, `quickActionsMenu`, `tooltipElement`
- **State tracking:** `currentSlideId`, `frameType`, `isShareIframe`
- **Performance tracking:** `loggedElements`, `injectionAttempts`

### CSS Conventions
- **Use high specificity selectors:** `#element-id:hover` over `.class:hover`
- **Apply `!important` strategically for third-party site integration**
- **Follow Google Material Design patterns:**
  - Border radius: `5px` for Google-style tooltips, `24px` for pill buttons
  - Font family: `'Google Sans', Roboto, Arial, sans-serif`
  - Google blue: `#1a73e8`
  - Tooltip styling: `#202124` background, `15px` padding
  - Hover background: `rgba(11, 87, 208, 0.09)`
  - Box shadow: `0 2px 10px rgba(0,0,0,0.2)` for tooltips
  - Pointer events: `none` for non-interactive tooltips

## Code Patterns

### Proactive Injection Strategy
```javascript
// Multi-layered injection approach for instant availability
function initializeQuickActionsIntegration() {
  // 1. Immediate scan on page load
  scanForExistingQuickActionsMenu();
  
  // 2. Delayed scan for late-loading menus
  setTimeout(() => {
    if (!state.quickActionsInjected) {
      scanForExistingQuickActionsMenu();
    }
  }, 1000);
  
  // 3. Event-driven detection as fallback
  setupQuickActionsMenuDetection();
}

// Immediate injection if menu already exists
function scanForExistingQuickActionsMenu() {
  const existingMenu = document.querySelector('.goog-menu[role="menu"]');
  if (existingMenu && !state.quickActionsInjected) {
    log('🔍 Found existing Quick Actions menu on page load');
    injectCurrentSlideOption(existingMenu);
  }
}
```

### Quick Actions Menu Integration
```javascript
// Preferred: Native menu integration with proactive injection
function setupQuickActionsMenuDetection() {
  document.addEventListener('click', (event) => {
    const quickActionsButton = event.target.closest('#scb-quick-actions-menu-button');
    if (quickActionsButton) {
      // Update slide ID when menu is accessed
      const newSlideId = getCurrentSlideId();
      if (newSlideId !== state.currentSlideId) {
        state.currentSlideId = newSlideId;
      }
      
      // Only inject if not already done
      if (!state.quickActionsInjected && !state.fallbackCheckDone) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omriariav) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
