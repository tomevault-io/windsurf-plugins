---
trigger: always_on
description: This file provides guidance to WARP (warp.dev) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project overview

- This is a small, static web app that implements the **Sighthound Savings Analyzer**: a multi-step wizard that collects camera and cost inputs, then computes an optimal Sighthound configuration and savings.
- The core runtime lives entirely in the browser: `index.html` (markup), `styles/savings-analyzer.css` (layout and theming), and `app.js` (behavior and calculations).
- There is **no bundler or framework** in use (no React/Vue/etc.); JavaScript runs as a single module script attached directly to `index.html`.
- A Node-based harness `test-run.js` uses `jsdom` to simulate the browser and exercise the wizard flow for regression testing.

The root `README.md` only names the project; there are no additional build or workflow details defined there.

## Common commands

All commands assume the project root as the working directory.

### Setup

- Install Node dependencies (only `jsdom` plus transitive deps):
  - `npm install`

### Running the app locally

There is no dedicated dev server or build step; the app is a static HTML/CSS/JS bundle.

- Quick local preview (using any static file server):
  - `npx serve .`
  - Then open the reported URL (typically `http://localhost:3000`) and navigate to `/index.html`.
- Alternatively, open `index.html` directly in a browser; prefer using an HTTP server if you run into module/CORS restrictions with `file://` URLs.

### Tests / harness

There is **no formal test runner configured in `package.json`**; the default `npm test` script just exits with an error placeholder. Instead, use the Node harness:

- Run the jsdom-based navigation harness:
  - `node test-run.js`
- This script will:
  - Load `index.html` into `jsdom`.
  - Wait for the app to signal that initialization completed via `window.__savings_init_done` (set in `app.js`).
  - Simulate typical user interactions (choosing a camera option, proceeding through steps, skipping step 3, navigating back and forth, toggling software selection) and log the active step IDs and button state to stdout.
- To focus on a specific scenario, temporarily comment out sections of `test-run.js` and re-run `node test-run.js`; there is no granular "single test" mechanism beyond editing this harness.

## High-level architecture

### Files and responsibilities

- `index.html`
  - Defines the multi-step wizard structure as five `.step` sections (`#step1`, `#step1b`, `#step2`, `#step3`, `#step4`, `#step5`) plus a `#results` section.
  - Uses semantic IDs (e.g., `continueStep2`, `backStep3`, `totalCamerasDisplay`, `setupGrid`, `costComparison`, `savingsCard`, `downloadPdf`) that are hard-wired into `app.js`.
  - Attaches the main behavior via `<script type="module" src="./app.js"></script>` and pulls in `styles/savings-analyzer.css`.

- `styles/savings-analyzer.css`
  - Contains a full, pre-generated stylesheet (Webflow-style reset + Sighthound-branded theme) and all layout/typography for the analyzer.
  - Defines visual states such as `.step.active`, `.node-status` variants, `.timeframe-btn.active`, and button styles used by `index.html`.
  - There is **no build pipeline** for CSS; this is the final stylesheet served to the browser.

- `app.js`
  - Owns **all interactive behavior and business logic** for the wizard and results, structured around a single mutable `state` object and a set of helper functions.
  - Attaches event listeners once (via `init()`), but uses some **delegated handlers** (notably a body-level click listener) to remain robust in different environments (browser vs. jsdom).
  - Exposes a `window.__savings_init_done` flag so external harnesses (like `test-run.js`) can detect when initialization has completed.

- `test-run.js`
  - Headless integration harness built on `jsdom`.
  - Loads `index.html`, applies workarounds for canvas and image APIs used by jsPDF, waits for the app to initialize, and then programmatically drives the UI.
  - Uses real `MouseEvent` and `change` events to stay close to real browser behavior.

### State and navigation model (`app.js`)

- **Global state**
  - `state` is a plain object holding user selections and derived settings: current step, camera type, ownership model, counts for standard/smart cameras, compute nodes, auto-add toggle, selected software options, current costs, billing frequency, and comparison timeframe.
  - This state is the single source of truth for both UI updates and calculations; all helper functions read/write from it.

- **Initialization lifecycle**
  - `init()` is idempotent (guarded by an internal `__savings_init_done` boolean) and can run safely even if triggered multiple times from different DOM lifecycle events.
  - Initialization sequence:
    - Attach all event handlers.
    - Derive initial cameras/nodes and selected software state.
    - Initialize the continue-button state for step 3.
    - Navigate to step 1.
    - Finally, set `window.__savings_init_done = true` to signal readiness to external tools.
  - `init()` is wired to all of: `DOMContentLoaded`, `window.load`, and a `setTimeout(init, 0)` if the document is already past `loading`, ensuring it runs in both real browsers and jsdom.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sighthoundinc/sighthound-savings-analyzer](https://github.com/sighthoundinc/sighthound-savings-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
