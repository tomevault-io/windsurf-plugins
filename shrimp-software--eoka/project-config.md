---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Build & Test

```bash
cargo build                          # Build library
cargo build --examples               # Build examples
cargo run --example basic            # Basic usage
cargo run --example detection_test   # Bot detection tests (sannysoft, browserleaks, etc.)
cargo run --example rebrowser_test   # Rebrowser bot detector test
cargo run --example detection_test -- --visible  # Visible browser
cargo run --example request_capture  # HTTP request capture demo
```

### Tests & lint

Unit/regression tests are pure logic — no browser required.

```bash
cargo test --lib                     # all unit + regression tests
cargo t                              # alias for the above
cargo test --lib session::           # one module
cargo test --lib test_cookie_header  # one test (substring match)
cargo test --lib -- --nocapture      # show test output

cargo fmt                            # format
cargo fmt --all --check              # verify formatting (CI)
cargo clippy --all-features -- -D warnings   # lint (CI-equivalent)
cargo lint                           # alias for the above
```

Pre-push gate (mirrors CI in `.github/workflows/ci.yml`):

```bash
cargo fmt --all --check && cargo lint && cargo test --lib
```

Note: `stealth::patcher::tests::test_find_chrome_returns_elf` fails locally only
when the system Chrome is a wrapper script (not an ELF). It's guarded by
`if let Ok(path) = find_chrome()`, so it passes in CI (no Chrome installed) and
on machines with a real Chrome. Skip it locally with
`cargo test --lib -- --skip test_find_chrome_returns_elf`.

## Architecture

```
src/
├── lib.rs              # Public API: Browser, Page, StealthConfig, Result
├── browser.rs          # Chrome launcher, stealth args
├── page.rs             # Page abstraction, Element, request capture
├── session.rs          # Cookie import/export
├── error.rs            # Error types (ElementNotVisible, RetryExhausted, etc.)
├── cdp/
│   ├── transport.rs    # WebSocket client + command filtering
│   ├── connection.rs   # Browser/Session CDP wrappers
│   └── types.rs        # Hand-written CDP types (~30 commands)
└── stealth/
    ├── evasions.rs     # 15 JavaScript injection scripts
    ├── patcher.rs      # Binary patching (Aho-Corasick)
    ├── human.rs        # Bezier curves, typing simulation
    └── fingerprint.rs  # User agent generation
```

## Public API Overview

### Browser
- `Browser::launch()` / `Browser::launch_with_config(config)`
- `Browser::launch_visible()` / `Browser::launch_debug()` - Common presets without manual config
- `Browser::launch_with(|config| { ... })` - Inline tweaks to default stealth config
- `browser.new_page(url)` - Create page and navigate
- `browser.tabs()` - List all open tabs (returns `Vec<TabInfo>`)
- `browser.activate_tab(id)` - Focus a tab
- `browser.close_tab(id)` - Close a specific tab
- `browser.close()`

### Page - Finding Elements
- `page.find(selector)` / `page.find_all(selector)` - By CSS selector
- `page.find_by_text(text)` - By visible text (prioritizes links/buttons)
- `page.find_by_text_match(text, TextMatch)` - With match strategy (Exact/Contains/StartsWith/EndsWith)
- `page.find_all_by_text(text)` - All elements with text
- `page.find_any(&[selectors])` - First matching selector
- `page.exists(selector)` / `page.text_exists(text)` - Check existence

### Page - Navigation
- `page.goto(url)` - Navigate to URL
- `page.goto_with_referrer(url, referrer)` - Navigate with custom Referer
- `page.goto_with_headers(url, headers)` - Navigate with custom HTTP headers
- `page.reload()` - Reload the page
- `page.back()` / `page.forward()` - History navigation

### Page - Clicking
- `page.click(selector)` / `page.human_click(selector)` - Standard click
- `page.click_at(x, y)` - Click at coordinates
- `page.click_by_text(text)` / `page.human_click_by_text(text)` - By text
- `page.try_click(selector)` - Returns `Ok(false)` if not found/visible
- `page.try_click_by_text(text)` / `page.try_human_click(selector)` / `page.try_human_click_by_text(text)`

### Page - Form Filling
- `page.fill(selector, value)` - Clear and type
- `page.human_fill(selector, value)` - Human-like clear and type
- `page.type_text(text)` - Type into focused element
- `page.type_into(selector, text)` - Type without clearing
- `page.human_type(selector, text)` - Human-like typing

### Page - Waiting
- `page.wait_for(selector, timeout)` - Wait for element in DOM
- `page.wait_for_visible(selector, timeout)` - Wait for element to be clickable
- `page.wait_for_hidden(selector, timeout)` - Wait for element to disappear
- `page.wait_for_any(&[selectors], timeout)` - Wait for any selector
- `page.wait_for_text(text, timeout)` - Wait for text to appear
- `page.wait_for_url_contains(pattern, timeout)` - Wait for URL pattern
- `page.wait_for_url_change(timeout)` - Wait for navigation
- `page.wait_for_network_idle(idle_ms, timeout)` - Wait for XHR/fetch to complete
- `page.wait(ms)` - Fixed delay

### Page - Info & Debug
- `page.url()` / `page.title()` / `page.content()` / `page.text()`
- `page.target_id()` - Get tab identifier (for multi-tab)
- `page.screenshot()` / `page.screenshot_jpeg(quality)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shrimp-software/eoka](https://github.com/shrimp-software/eoka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
