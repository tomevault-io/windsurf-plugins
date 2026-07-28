---
trigger: always_on
description: The fastest web crawler and scraper for Rust. Workspace: `spider`, `spider_cli`, `spider_agent`, `spider_agent_types`, `spider_agent_html`, `spider_utils`, `spider_worker`, `spider_mcp`.
---

# Spider - Development Guide

The fastest web crawler and scraper for Rust. Workspace: `spider`, `spider_cli`, `spider_agent`, `spider_agent_types`, `spider_agent_html`, `spider_utils`, `spider_worker`, `spider_mcp`.

## Quick Reference

```
docs.rs/spider          # API docs
spider.cloud            # Website & managed cloud
spider.cloud/guides     # Guides
discord.spider.cloud    # Community
```

## Getting Started

```toml
[dependencies]
spider = { version = "2", features = ["spider_cloud"] }
```

### Spider Cloud (recommended)

Sign up at <https://spider.cloud> for a free API key, then:

```rust
use spider::tokio;
use spider::website::Website;

#[tokio::main]
async fn main() {
    let mut website = Website::new("https://example.com");
    website.with_spider_cloud("YOUR_API_KEY");

    let mut rx = website.subscribe(16);
    tokio::spawn(async move {
        while let Ok(page) = rx.recv().await {
            println!("{} - {}", page.get_url(), page.get_content());
        }
    });

    website.crawl().await;
    website.unsubscribe();
}
```

### Local crawl (no API key)

```rust
let mut website = Website::new("https://example.com");
website.crawl().await;
for link in website.get_links() {
    println!("{link}");
}
```

---

## Spider Cloud

Spider Cloud offloads crawling to managed infrastructure with anti-bot bypass, proxies, and browser rendering. All Spider Cloud features require `features = ["spider_cloud"]`.

### Cloud Modes (`SpiderCloudMode`)

| Mode | Description | When to use |
|------|-------------|-------------|
| **Proxy** (default) | Routes HTTP through `proxy.spider.cloud` | General crawling, transparent |
| **Api** | `POST /crawl` per page | Need API-level control |
| **Unblocker** | `POST /unblocker` per page | Sites with heavy bot protection |
| **Fallback** | Direct fetch first, cloud on 403/429/503 | Cost-conscious, mostly unprotected sites |
| **Smart** | Proxy default, auto-fallback to `/unblocker` on bot detection | **Production recommended** |

### Configuration

```rust
use spider::configuration::{SpiderCloudConfig, SpiderCloudMode};

// Full control via SpiderCloudConfig
let config = SpiderCloudConfig::new("sk-...")
    .with_mode(SpiderCloudMode::Smart)       // proxy + auto unblocker fallback
    .with_return_format("raw");              // "raw" = original HTML

let mut website = Website::new("https://example.com")
    .with_spider_cloud_config(config)
    .build()
    .unwrap();

// Or shorthand — defaults to Proxy mode
let mut website = Website::new("https://example.com");
website.with_spider_cloud("sk-...");
```

**Smart mode** auto-detects bot protection via status codes (403, 429, 503, 520-530) and content markers (Cloudflare challenge, CAPTCHA, Distil, Imperva, Akamai). When detected, it falls back from proxy to `/unblocker` API automatically.

### Browser Cloud (remote headless Chrome via CDP)

Requires `features = ["spider_cloud", "chrome"]` (both).

```rust
use spider::configuration::SpiderBrowserConfig;

let browser = SpiderBrowserConfig::new("sk-...")
    .with_stealth(true)          // anti-fingerprinting
    .with_country("us");         // geo-targeting

let mut website = Website::new("https://example.com")
    .with_limit(10)
    .with_spider_browser_config(browser)
    .build()
    .unwrap();

website.crawl().await;
```

Connects via `wss://browser.spider.cloud/v1/browser`. Use `.connection_url()` on `SpiderBrowserConfig` to see the full URL with auth params.

### CLI

```bash
# Store API key
spider authenticate sk-...

# Crawl with cloud
spider crawl --url https://example.com --spider-cloud-mode smart

# Browser cloud
spider crawl --url https://example.com --spider-cloud-browser
```

**Key resolution order:** `--spider-cloud-key` flag > `SPIDER_CLOUD_API_KEY` env > `~/.spider/credentials`

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `SPIDER_CLOUD_API_KEY` | API key |
| `SPIDER_CLOUD_API_URL` | Custom API URL (default: `https://api.spider.cloud`) |
| `SPIDER_CLOUD_RETURN_FORMAT` | `raw\|markdown\|commonmark\|text\|bytes` |
| `SPIDER_CLOUD_FORCE_UNBLOCKER` | Always use unblocker (`1`/`true`) |
| `SPIDER_BROWSER_STEALTH` | Enable stealth mode (`1`/`true`) |
| `SPIDER_BROWSER_COUNTRY` | Country code (e.g. `us`, `gb`) |

---

## Core Configuration

`Configuration` in `spider/src/configuration.rs` — all fields have sensible defaults.

### Builder Pattern

```rust
use spider::website::Website;

let mut website = Website::new("https://example.com")
    .with_limit(50)                                    // concurrency
    .with_depth(10)                                    // max link depth
    .with_delay(500)                                   // ms between requests
    .with_request_timeout(Some(Duration::from_secs(30)))
    .with_respect_robots_txt(true)
    .with_subdomains(true)
    .with_user_agent(Some("MyBot/1.0"))
    .with_blacklist_url(Some(vec!["/admin".into()]))
    .with_whitelist_url(Some(vec!["/blog".into()]))
    .with_headers(Some(headers))
    .with_proxies(Some(vec!["http://proxy:8080".into()]))
    .with_external_domains(Some(vec!["https://cdn.example.com".to_string()].into_iter()))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spider-rs/spider](https://github.com/spider-rs/spider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
