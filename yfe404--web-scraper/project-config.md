---
trigger: always_on
description: This skill activates for web scraping and Actor development. It proactively discovers APIs via traffic interception, recommends optimal strategy (traffic interception/sitemap/API/DOM scraping/hybrid), and implements iteratively. For production, it guides TypeScript Actor creation via Apify CLI.
---


# Web Scraping with Intelligent Strategy Selection

## When This Skill Activates

Activate automatically when user requests:
- "Scrape [website]"
- "Extract data from [site]"
- "Get product information from [URL]"
- "Find all links/pages on [site]"
- "I'm getting blocked" or "Getting 403 errors" (loads `strategies/anti-blocking.md`)
- "Make this an Apify Actor" (loads `apify/` subdirectory)
- "Productionize this scraper"

## Input Parsing

Determine reconnaissance depth from user request:

| User Says | Mode | Phases Run |
|-----------|------|------------|
| "quick recon", "just check", "what framework" | Quick | Phase 0 only |
| "scrape X", "extract data from X" (default) | Standard | Phases 0-3 + 5, Phase 4 only if protection signals detected |
| "full recon", "deep scan", "production scraping" | Full | All phases (0-5) including protection testing |

Default is Standard mode. Escalate to Full if protection signals appear during any phase.

## Adaptive Reconnaissance Workflow

This skill uses an adaptive phased workflow with quality gates. Each gate asks **"Do I have enough?"** — continue only when the answer is no.

**See**: `strategies/framework-signatures.md` for framework detection tables referenced throughout.

### Phase 0: QUICK ASSESSMENT (curl, no browser)

Gather maximum intelligence with minimum cost — a single HTTP request.

**Step 0a: Fetch raw HTML and headers**
```bash
curl -s -D- -L "https://target.com/page" -o response.html
```

**Step 0b: Check response headers**
- Match headers against `strategies/framework-signatures.md` → Response Header Signatures table
- Note `Server`, `X-Powered-By`, `X-Shopify-Stage`, `Set-Cookie` (protection markers)
- Check HTTP status code (200 = accessible, 403 = protected, 3xx = redirects)

**Step 0c: Check Known Major Sites table**
- Match domain against `strategies/framework-signatures.md` → Known Major Sites
- If matched: use the specified data strategy, skip generic pattern scanning

**Step 0d: Detect framework from HTML**
- Search raw HTML for signatures in `strategies/framework-signatures.md` → HTML Signatures table
- Look for `__NEXT_DATA__`, `__NUXT__`, `ld+json`, `/wp-content/`, `data-reactroot`

**Step 0e: Search for target data points**
- For each data point the user wants: search raw HTML for that content
- Track which data points are found vs missing
- Check for sitemaps: `curl -s https://[site]/robots.txt | grep -i Sitemap`

**Step 0f: Note protection signals**
- 403/503 status, Cloudflare challenge HTML, CAPTCHA elements, `cf-ray` header
- Record for Phase 4 decision

**See**: `strategies/cheerio-vs-browser-test.md` for the Cheerio viability assessment

> **QUALITY GATE A**: All target data points found in raw HTML + no protection signals?
> → YES: Skip to Phase 3 (Validate Findings). No browser needed.
> → NO: Continue to Phase 1.

### Phase 1: BROWSER RECONNAISSANCE (only if Phase 0 needs it)

Launch browser only for data points missing from raw HTML or when JavaScript rendering is required.

**Step 1a: Initialize browser session**
- `proxy_start()` → Start traffic interception proxy
- `interceptor_chrome_launch(url, stealthMode: true)` → Launch Chrome with anti-detection
- `interceptor_chrome_devtools_attach(target_id)` → Attach DevTools bridge
- `interceptor_chrome_devtools_screenshot()` → Capture visual state

**Step 1b: Capture traffic and rendered DOM**
- `proxy_list_traffic()` → Review all traffic from page load
- `proxy_search_traffic(query: "application/json")` → Find JSON responses
- `interceptor_chrome_devtools_list_network(resource_types: ["xhr", "fetch"])` → XHR/fetch calls
- `interceptor_chrome_devtools_snapshot()` → Accessibility tree (rendered DOM)

**Step 1c: Search rendered DOM for missing data points**
- For each data point NOT found in Phase 0: search rendered DOM
- Use framework-specific search strategy from `strategies/framework-signatures.md` → Framework → Search Strategy table
- Only search patterns relevant to the detected framework

**Step 1d: Inspect discovered endpoints**
- `proxy_get_exchange(exchange_id)` → Full request/response for promising endpoints
- Document: method, headers, auth, response structure, pagination

> **QUALITY GATE B**: All target data points now covered (raw HTML + rendered DOM + traffic)?
> → YES: Skip to Phase 3 (Validate Findings). No deep scan needed.
> → NO: Continue to Phase 2 for missing data points only.

### Phase 2: DEEP SCAN (only for missing data points)

Targeted investigation for data points not yet found. Only search for what's missing.

**Step 2a: Test interactions for missing data**
- `proxy_clear_traffic()` before each action → Isolate API calls
- `humanizer_click(target_id, selector)` → Trigger dynamic content loads
- `humanizer_scroll(target_id, direction, amount)` → Trigger lazy loading / infinite scroll
- `humanizer_idle(target_id, duration_ms)` → Wait for delayed content
- After each action: `proxy_list_traffic()` → Check for new API calls

**Step 2b: Sniff APIs (framework-aware)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yfe404/web-scraper](https://github.com/yfe404/web-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
