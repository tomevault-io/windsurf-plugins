---
trigger: always_on
description: > **Keep this file up to date.** When you make changes that affect architecture, crawl behavior, environment variables, or testing considerations described here, update the relevant section in the same commit.
---

# Oobee Developer Guide

> **Keep this file up to date.** When you make changes that affect architecture, crawl behavior, environment variables, or testing considerations described here, update the relevant section in the same commit.

Oobee is a web accessibility scanner that crawls websites and runs axe-core + custom checks against each page, producing HTML/PDF/CSV/JSON reports.

## Architecture Overview

```
User Input (CLI / npm API)
    ↓
combine.ts (orchestrator)
    ↓ routes by ScannerTypes
Crawler (crawlDomain / crawlSitemap / crawlIntelligentSitemap / crawlLocalFile / runCustom)
    ↓ uses Crawlee PlaywrightCrawler
Page Handler (axe-core injection + custom checks)
    ↓ writes per-page JSON to Crawlee dataset
generateArtifacts() in mergeAxeResults.ts
    ↓ reads dataset, aggregates, renders templates
Reports (HTML, PDF, CSV, JSON, sitemap.xml)
```

## Entry Points

| Entry | File | Purpose |
|-------|------|---------|
| CLI | `src/cli.ts` | yargs-based CLI, calls `combineRun()` |
| Interactive CLI | `src/index.ts` | Inquirer prompts, calls `combineRun()` |
| npm API | `src/npmIndex.ts` | Programmatic `init()` for external consumers |
| Orchestrator | `src/combine.ts` | Routes scan type, manages lifecycle, calls `generateArtifacts()` |

## Scanner Types

| Type | File | Behavior |
|------|------|----------|
| `Website` | `src/crawlers/crawlDomain.ts` | Domain crawl, discovers links from pages |
| `Sitemap` | `src/crawlers/crawlSitemap.ts` | Fetches URLs from sitemap XML |
| `Intelligent` | `src/crawlers/crawlIntelligentSitemap.ts` | Discovers sitemap via robots.txt, crawls it, then supplements with domain crawl |
| `LocalFile` | `src/crawlers/crawlLocalFile.ts` | Scans local HTML/PDF files via file:// |
| `Custom` | `src/crawlers/runCustom.ts` | User-driven flow (manual navigation in browser) |

## Key Files

### Constants & Configuration

- **`src/constants/constants.ts`** — Enums (`ScannerTypes`, `BrowserTypes`, `FileTypes`, `RuleFlags`), browser data dir paths, sitemap path list, WCAG mappings, shared mutable state (`robotsTxtUrls`, `sitemapFetchedLinks`, `userDataDirectory`, `launcher`)
- **`src/constants/common.ts`** — URL validation (`checkUrl`), browser launch options (`getPlaywrightLaunchOptions`), sitemap parsing (`getLinksFromSitemap`, `getSitemapsFromRobotsTxt`), robots.txt handling, browser selection (`getBrowserToRun`), user-agent initialization (`initModifiedUserAgent`)

### Crawlers

All crawlers use Crawlee's `PlaywrightCrawler` with:
- `maxRequestsPerCrawl: Infinity` (Crawlee's internal limit disabled)
- Manual stop when `urlsCrawled.scanned.length >= maxRequestsPerCrawl` (counts only successful scans)
- `retryOnBlocked: true`
- `useFingerprints: false`

### Report Generation

- **`src/mergeAxeResults.ts`** — Main `generateArtifacts()` function, reads Crawlee dataset, builds `allIssues` object, generates all output formats
- **`src/mergeAxeResults/`** — Sub-modules: `jsonArtifacts.ts` (JSON+base64), `writeCsv.ts`, `writeSitemap.ts`, `scanPages.ts`, `itemsStore.ts`, `types.ts`
- **`src/static/ejs/`** — EJS templates for HTML report and PDF summary

## Browser Handling

### Selection Priority

`getBrowserToRun()` in `common.ts` resolves the browser:
- If no preference specified: defaults to Chrome on Windows/macOS, Chromium on Linux
- Fallback chains:
  - **macOS**: Chrome → webkit
  - **Windows**: Chrome → Edge → error
  - **Linux**: Chrome → Chromium (bundled by Playwright)
- When `chromium` is specified: uses Playwright's bundled Chromium with no channel

### Launch Options

`getPlaywrightLaunchOptions()` builds Playwright launch config:
- Headless mode from `process.env.CRAWLEE_HEADLESS`
- Docker detection (`/.dockerenv`): adds `--disable-gpu`, `--no-sandbox`, `--disable-dev-shm-usage`
- Proxy support (manual, PAC, or none) via `getProxyInfo()`
- Channel set from browser name (undefined for chromium = bundled)
- `--mute-audio` is added by default in both headless and headful modes, but must be disabled for `customFlow` by calling `getPlaywrightLaunchOptions(browser, { includeMuteAudio: false })`

### User-Agent

`initModifiedUserAgent()` detects the default UA, replaces `HeadlessChrome` with `Chrome`, stores in `process.env.OOBEE_USER_AGENT`. This must be called before any browser context that talks to remote servers in headless mode, or bot-blocking WAFs will reject requests.

Contexts that need `userAgent: process.env.OOBEE_USER_AGENT`:
- `getRobotsTxtViaPlaywright()` — robots.txt fetching
- `findSitemap()` in `crawlIntelligentSitemap.ts` — sitemap path probing
- `getDataUsingPlaywright()` in `getLinksFromSitemap()` — sitemap XML content fetching
- `checkUrl()` — main URL validation context (already handled)
- Crawlee crawler contexts in `crawlDomain`/`crawlSitemap` — UA set via `preLaunchHooks` in `getPreLaunchHook()`

### Headless vs Headful

- Docker/Linux: always headless (`CRAWLEE_HEADLESS=1`)
- macOS CLI: typically headful (`CRAWLEE_HEADLESS=0`) unless user opts in
- Headful mode uses ephemeral contexts (no `userDataDir`) to avoid "Browser window not found" errors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GovTechSG/oobee](https://github.com/GovTechSG/oobee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
