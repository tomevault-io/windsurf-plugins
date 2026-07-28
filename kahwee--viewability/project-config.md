---
trigger: always_on
description: This document provides technical details about the `viewability` package for AI assistants like Claude to better understand and work with this codebase.
---

# CLAUDE.md - Technical Documentation for AI Assistants

This document provides technical details about the `viewability` package for AI assistants like Claude to better understand and work with this codebase.

## Package Overview

**viewability** is a lightweight browser library for measuring DOM element visibility within the viewport. It provides precise percentage-based measurements of how much of an element is visible, along with descriptive state information.

### Core Purpose

- Measure vertical and horizontal visibility of DOM elements
- Return precise visibility percentages (0-1 scale)
- Provide descriptive state strings explaining element position
- Enable use cases like ad viewability tracking, lazy loading, and scroll animations

### Key Characteristics

- **Zero dependencies**: Completely standalone, no external libraries
- **Small footprint**: Minimal code, fast execution
- **Browser-based**: Uses standard DOM APIs (`getBoundingClientRect`, `window.innerWidth/Height`)
- **CommonJS modules**: Standard Node.js module structure
- **UMD build available**: Can be used in browsers via CDN

## Architecture

### File Structure

```
viewability/
├── viewability.js          # Main entry point, exports all functions
├── vertical.js             # Vertical viewability calculation
├── horizontal.js           # Horizontal viewability calculation
├── isElementOnScreen.js    # Convenience boolean checker
├── webpack.config.js       # Build configuration for UMD bundle
├── karma.conf.js          # Test runner configuration
└── tests/                 # Test suite
    ├── vertical-spec.js
    ├── horizontal-spec.js
    └── isElementOnScreen-spec.js
```

### Module Design

Each function is in its own file for modularity. Users can import just what they need:

```js
// Import only vertical checking
const vertical = require('viewability/vertical');

// Or import the main package with all functions
const viewability = require('viewability');
```

## Implementation Details

### vertical.js

**Purpose**: Calculates vertical visibility of an element within the viewport.

**Algorithm**:
1. Get viewport height: `window.innerHeight`
2. Get element position using `getBoundingClientRect()`:
   - `top`: Distance from top of viewport to top of element
   - `bottom`: Distance from top of viewport to bottom of element
3. Calculate element height: `bottom - top`
4. Determine visibility state and percentage:

**States**:
- `EL_IS_BELOW_VIEW`: Element is completely below viewport (top > windowHeight)
- `EL_IS_ABOVE_VIEW`: Element is completely above viewport (bottom <= 0)
- `EL_IS_WITHIN_VERTICAL_VIEW`: Element is fully visible (top >= 0 && bottom <= windowHeight)
- `EL_BOTTOM_AND_TOP_TRUNCATED`: Element is taller than viewport, both ends cut off
- `EL_TOP_TRUNCATED`: Top portion is above viewport, bottom is visible
- `EL_BOTTOM_TRUNCATED`: Bottom portion is below viewport, top is visible
- `EL_IS_NOT_WITHIN_VIEW`: Generic error state

**Return Value**:
```typescript
{
  value: number,  // 0 to 1, representing visible percentage
  state: string   // One of the states above
}
```

### horizontal.js

**Purpose**: Calculates horizontal visibility of an element within the viewport.

**Algorithm**: Same as vertical but uses:
- `window.innerWidth` instead of `window.innerHeight`
- `left` and `right` from `getBoundingClientRect()`
- Element width: `right - left`

**States**:
- `EL_IS_TOO_RIGHT`: Element is completely right of viewport
- `EL_IS_TOO_LEFT`: Element is completely left of viewport
- `EL_IS_WITHIN_HORIZONTAL_VIEW`: Element is fully visible
- `EL_LEFT_AND_RIGHT_TRUNCATED`: Element is wider than viewport
- `EL_LEFT_TRUNCATED`: Left portion is outside viewport
- `EL_RIGHT_TRUNCATED`: Right portion is outside viewport
- `EL_IS_NOT_WITHIN_VIEW`: Generic error state

### isElementOnScreen.js

**Purpose**: Convenience function for boolean visibility check.

**Algorithm**:
```js
function isElementOnScreen(el, full) {
  if (full) {
    // Check if 100% visible (both dimensions)
    return vertical(el).value * horizontal(el).value === 1;
  } else {
    // Check if any part is visible
    return vertical(el).value * horizontal(el).value > 0;
  }
}
```

**Parameters**:
- `el` (HTMLElement): Element to check
- `full` (boolean): If true, requires 100% visibility; if false/omitted, any visibility counts

**Note**: Multiplies vertical and horizontal values to get 2D visibility.

## API Reference

### Main Module (viewability.js)

```js
const viewability = require('viewability');

// Exports:
{
  vertical: Function,      // From vertical.js
  horizontal: Function,    // From horizontal.js
  isElementOnScreen: Function  // From isElementOnScreen.js
}
```

### Individual Functions

#### vertical(element: HTMLElement): ViewabilityResult

Returns vertical visibility information.

```js
const vertical = require('viewability/vertical');
const result = vertical(document.getElementById('my-element'));
// { value: 0.75, state: 'EL_BOTTOM_TRUNCATED' }
```

#### horizontal(element: HTMLElement): ViewabilityResult

Returns horizontal visibility information.

```js
const horizontal = require('viewability/horizontal');

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kahwee/viewability](https://github.com/kahwee/viewability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
