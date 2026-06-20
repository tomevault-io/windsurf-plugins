---
trigger: always_on
description: >
---


# Autonomous Browser Control (AI Cowork)

Full autonomous web agent — navigate, extract, interact, and chain multi-step workflows across any website, powered by Playwright with production-grade stealth.

## When to Use This Skill

Use this skill when the user asks you to:

- **Browse or navigate** websites autonomously
- **Extract data** from web pages (tables, text, links, images, structured content)
- **Automate web tasks** — fill forms, click buttons, submit data, upload files
- **Monitor websites** for changes, new content, or price drops
- **Chain multi-step workflows** across multiple sites (e.g., search → compare → extract → report)
- **Log into services** and perform authenticated actions
- **Take screenshots** or capture visual state of pages
- **Interact with dynamic content** — SPAs, infinite scroll, lazy-loaded elements, modals
- **Simulate human behavior** on websites for testing or interaction

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│              AUTONOMOUS BROWSER AGENT                │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │ PLANNER  │→ │ EXECUTOR │→ │ RESULT PROCESSOR  │  │
│  │          │  │          │  │                    │  │
│  │ Decompose│  │ Playwright│  │ Parse + Structure │  │
│  │ tasks    │  │ actions  │  │ data              │  │
│  └──────────┘  └──────────┘  └───────────────────┘  │
│       ↑              │                    │          │
│       │         ┌────┴─────┐              │          │
│       │         │ STEALTH  │              ↓          │
│       │         │ LAYER    │      ┌──────────────┐   │
│       │         └──────────┘      │ OUTPUT       │   │
│       └───────── FEEDBACK ←───────│ JSON/CSV/Log │   │
│                  LOOP             └──────────────┘   │
└─────────────────────────────────────────────────────┘
```

The agent operates in 6 phases:
1. **Planning** — Decompose user intent into discrete browser actions
2. **Stealth Setup** — Configure anti-detection before any navigation
3. **Execution** — Perform browser actions with retry and error recovery
4. **Extraction** — Parse and structure collected data
5. **Validation** — Verify results, retry failed steps
6. **Reporting** — Deliver structured output with logs and screenshots

---

## Instructions

### Phase 1: Planning — Task Decomposition

Before launching the browser, decompose the user's request into an ordered action plan.

**Step 1 — Parse user intent**

Classify the request into one or more action types:

| Action Type | Description | Example |
|-------------|-------------|---------|
| `navigate` | Go to a URL or search for a site | "Go to GitHub" |
| `extract` | Pull data from a page | "Get all product prices" |
| `interact` | Click, type, select, scroll | "Fill out the contact form" |
| `authenticate` | Log into a service | "Log into my dashboard" |
| `monitor` | Watch for changes over time | "Alert me when price drops" |
| `capture` | Screenshot or save page state | "Screenshot the results page" |
| `chain` | Multi-step across pages/sites | "Search, compare, then buy" |

**Step 2 — Build action sequence**

Create an ordered list of atomic browser actions:

```
Action Plan: <task_description>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 1: [navigate] → https://example.com
Step 2: [interact] → Click "Login" button
Step 3: [interact] → Fill email field with <value>
Step 4: [interact] → Fill password field with <value>
Step 5: [interact] → Click "Submit"
Step 6: [extract]  → Get dashboard data table
Step 7: [capture]  → Screenshot final state
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Estimated steps: 7
Risk factors: Auth required, possible CAPTCHA
Stealth level: HIGH
```

**Step 3 — Present plan to user for confirmation before execution.**

---

### Phase 2: Stealth Setup — Anti-Detection Layer

**CRITICAL: Always configure stealth BEFORE the first navigation.**

#### 2A — Browser Context Configuration

```javascript
const { chromium } = require('playwright');

async function createStealthContext(options = {}) {
  const {
    proxy = null,
    locale = 'en-US',
    timezone = 'America/New_York',
    viewport = null,
    geolocation = null,
  } = options;

  // Randomize viewport from common resolutions
  const viewports = [
    { width: 1920, height: 1080 },
    { width: 1366, height: 768 },
    { width: 1536, height: 864 },
    { width: 1440, height: 900 },
    { width: 1280, height: 720 },
    { width: 2560, height: 1440 },
  ];
  const selectedViewport = viewport || viewports[Math.floor(Math.random() * viewports.length)];

  // Randomize user agent from real-world common agents
  const userAgents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:123.0) Gecko/20100101 Firefox/123.0',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.3 Safari/605.1.15',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BizraInfo/bizra-data-lake](https://github.com/BizraInfo/bizra-data-lake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
