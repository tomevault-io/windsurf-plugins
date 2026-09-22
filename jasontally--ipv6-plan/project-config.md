---
trigger: always_on
description: This file contains guidance for LLM agents working on the IPv6 Subnet Planner codebase.
---

# AGENTS.md - LLM Working Instructions

This file contains guidance for LLM agents working on the IPv6 Subnet Planner codebase.

**Copyright (c) 2025 Jason Tally and contributors** - SPDX-License-Identifier: MIT

## Project Overview

This is a **single-file web application** (HTML/CSS/JS) for planning IPv6 address space allocations. The app has:

- No build process
- No external dependencies
- Works entirely in the browser
- State persisted in URL hash

**Update:** The codebase now has testing infrastructure with Vitest (unit tests) and Playwright (E2E tests). See the "Testing" section below for details.

## File Structure

```
v6calc/
├── index.html                    # Single file containing HTML and CSS
├── app.js                        # Application JavaScript (extracted from HTML)
├── README.md                     # User-facing documentation
├── ARCHITECTURE.md               # Technical architecture and algorithms
├── AGENTS.md                     # This file - LLM working instructions
├── package.json                  # Project configuration and test scripts
├── vitest.config.js              # Vitest configuration for unit tests
├── playwright.config.js            # Playwright configuration for E2E tests
└── tests/
    ├── ipv6.test.js            # Unit tests for IPv6 utilities
    ├── subnet-tree.test.js       # Unit tests for tree operations
    ├── state.test.js            # Unit tests for state management
    └── e2e/
        ├── split-join.spec.js    # E2E tests for split/join
        ├── url-export.spec.js     # E2E tests for URL/CSV
        ├── initialization.spec.js  # E2E tests for app initialization
        ├── error-scenarios.spec.js  # E2E tests for error handling
        ├── accessibility.spec.js  # E2E tests for accessibility
        └── stress.spec.js       # E2E tests for large trees
```

## Working with This Codebase

### Before Making Changes

1.  **Read the relevant documentation:**
    - `ARCHITECTURE.md` - Understanding data structures and algorithms
    - This file (`AGENTS.md`) - Code conventions and testing

2.  **Understand the single-file constraint:**
    - All code stays in `index.html`
    - Do not split into separate files unless explicitly requested
    - Do not introduce build tools or bundlers 3. **Verify the change is safe:**
      - Test the application manually after changes
      - Run automated tests: `npm test` and `npm run test:e2e`

      ### Testing Philosophy

**When to Use Unit Tests vs E2E Tests:**

**Unit Tests are ideal for:**

- Pure functions with deterministic inputs/outputs
- Business logic algorithms (IPv6 parsing, subnet math)
- Data structure operations (tree manipulation)
- Error handling for invalid inputs
- Functions that don't depend on browser APIs

**E2E Tests are required for:**

- Browser-specific APIs (Compression Streams, Clipboard, File handling)
- DOM interactions and UI behavior
- Cross-browser compatibility testing
- Integration of multiple systems working together
- Features that rely on browser environment (URL sharing, downloads)

**Anti-Pattern to Avoid:**

- **Never modify production code to work around unit test environment limitations**
- If a unit test fails due to missing browser APIs, move the test to E2E
- Don't add mocks for browser-specific functionality - test in real browsers

**Example:** The compression utilities were initially tested with unit tests, but Node.js lacks the `blob.arrayBuffer()` Web API. Instead of modifying the compression code to work in Node.js, the tests were moved to E2E where they test real browser behavior.

### Writing Tests

**Unit Tests Location:** `tests/*.test.js`

- `ipv6.test.js` - Tests for parseIPv6, formatIPv6, applyPrefix, compareCIDR
- `subnet-tree.test.js` - Tests for splitSubnet, joinSubnet, getSubnetNode, isSplit
- `state.test.js` - Tests for saveState, loadState, loadNetwork
- `create-intermediate-level.test.js` - Tests for createIntermediateLevel function
- `create-intermediate-levels.test.js` - Tests for createIntermediateLevels function
- `create-intermediate-extra.test.js` - Tests for additional intermediate level scenarios
- `delete-descendants.test.js` - Tests for deleteDescendants function
- `nibble-boundaries.test.js` - Tests for getNibbleBoundaries function
- `auto-split-target.test.js` - Tests for getAutoSplitTarget (4-bit and 8-bit auto step)

**E2E Tests Location:** `tests/e2e/*.spec.js`

- `split-join.spec.js` - Tests for split, join operations, UI interactions
- `url-export.spec.js` - Tests for URL sharing, CSV export, download handling
- `initialization.spec.js` - E2E tests for app initialization
- `error-scenarios.spec.js` - E2E tests for error handling
- `accessibility.spec.js` - E2E tests for accessibility
- `stress.spec.js` - E2E tests for large trees
- `color-picker.spec.js` - E2E tests for color picker functionality
- `subnet-math.spec.js` - E2E tests for subnet math visual verification (split displays, sequential addresses, format correctness)
- `auto-split-bits.spec.js` - E2E tests for the 4-bit/8-bit auto-split step selector and URL persistence

### Accessibility Requirements

**All UI changes must meet WCAG 2.1 Level AA standards.**

**Mandatory for all interactive elements:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasontally/IPv6-Plan](https://github.com/jasontally/IPv6-Plan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
