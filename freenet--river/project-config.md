---
trigger: always_on
description: - Use `freenet local` for routine River UI development and manual testing. Local mode exercises the app without spinning up multiple network peers.
---

# Claude Guide for River

## Freenet Node Operation
- Use `freenet local` for routine River UI development and manual testing. Local mode exercises the app without spinning up multiple network peers.
- Use `freenet network` only when validating peer-to-peer sync or multi-node behaviour. Document the scenario when you switch modes.

## Project Overview
River is a decentralized group chat application built on Freenet and consists of:
- Rust + Dioxus web UI compiled to WebAssembly
- Smart contracts (room and web container) deployed on Freenet
- Delegates that execute contract logic and perform background tasks
- Shared `common/` crate with data types and crypto helpers used across UI and contracts

## Essential Commands

### Development
```bash
cargo make dev-example             # UI with example data, no Freenet connection
cargo make dev                     # Standard development server
cargo make build                   # Full release build
cargo make build-ui                # UI artifacts only
cargo make build-ui-example-no-sync# UI build with example data and no sync
```

### Testing
```bash
cargo make test                    # Full workspace tests
cargo make test-room-contract
cargo make test-web-container
cargo make test-common
cargo make test-chat-delegate
cargo make test-web-container-integration
cargo test -p river-ui --bins      # river-ui crate native unit tests (CI-gated)
```

### Local UI Testing with dx serve

For rapid UI iteration without publishing to Freenet:

```bash
# From the ui/ directory
cd ui

# Local only (127.0.0.1)
dx serve --port 8082 --features example-data,no-sync

# Accessible from other machines (0.0.0.0)
dx serve --port 8082 --addr 0.0.0.0 --features example-data,no-sync
```

**Features:**
- `example-data` - Populates UI with sample rooms, members, messages, and reactions
- `no-sync` - Disables Freenet sync (no WebSocket connection required)

**Tips:**
- dx serve auto-rebuilds on file changes, but sometimes needs manual restart
- Check `/tmp/dx-serve-new.log` for build errors if UI doesn't update
- Use `--addr 0.0.0.0` when testing from remote machines (e.g., technic → nova)
- Example data includes reactions on messages for testing the emoji picker UI

### Playwright UI Tests (REQUIRED before publishing)

**Always run Playwright tests before publishing to Freenet.** Republishing takes minutes, so catch layout issues locally first.

```bash
# One-time setup: install browsers
cargo make test-ui-playwright-setup

# Build UI with example data (no Freenet connection needed)
cargo make build-ui-example-no-sync

# Serve built files (do NOT use dx serve — it auto-rebuilds and can serve stale content)
cd target/dx/river-ui/release/web/public && python3 -m http.server 8082 &

# Run all tests across Chromium, Firefox, WebKit, mobile Chrome, mobile Safari
cd ui/tests && npx playwright test

# Run specific browser or test
npx playwright test --project=chromium
npx playwright test --project=webkit --grep "iframe"
npx playwright test --project=mobile-safari --grep "Mobile"
```

**Test coverage:**
- Desktop 1280px: 3-column layout, no overflow
- Tablet 768px: narrower sidebars via CSS clamp
- Breakpoint 767px: mobile mode (single panel)
- Mobile 480px: view switching (hamburger, members, back buttons)
- Mobile 320px: small screen readability
- Desktop recovery after mobile resize
- Sandboxed iframe embedding (matching Freenet gateway)

**Important Tailwind v4 note:** The `@source "../src/**/*.rs"` directive in `ui/assets/tailwind.css` is REQUIRED. Without it, Tailwind v4 won't scan Rust files for class names, and responsive utilities like `md:flex` won't be generated.

**Two test directories exist:**
- `ui/tests/` — Layout/visual tests against `dx build` with example data (runs in CI)
- `e2e-test/` — Integration tests against a real Freenet node (manual)

### Interactive Playwright MCP (for debugging and verification)

The Playwright MCP plugin is enabled in `.claude/settings.local.json`. Use it
to interactively test the UI against a running local node — no manual browser
needed.

**Testing against example data (no Freenet node required):**
```bash
# Build and serve with example data
cargo make build-ui-example-no-sync
cd target/dx/river-ui/release/web/public && python3 -m http.server 8082 &
```
Then use Playwright MCP tools:
1. `browser_navigate` → `http://127.0.0.1:8082/`
2. `browser_snapshot` → inspect DOM state, verify layout
3. `browser_click` / `browser_fill_form` → interact with UI elements
4. `browser_console_messages` → check for WASM panics or JS errors

**Testing against a local Freenet node (full integration):**
```bash
# Publish to local node first
./scripts/local-republish.sh
# Script outputs the URL, e.g.:
#   http://127.0.0.1:7510/v1/contract/web/{CONTRACT_ID}/
```
Then use Playwright MCP tools to navigate to the published URL. This tests
the full stack: WASM ↔ WebSocket ↔ Freenet node ↔ contract/delegate.

**Common verification tasks with Playwright MCP:**
- **After UI changes**: Navigate, take snapshot, verify layout renders correctly
- **After message send fixes**: Fill message input, click send, verify message appears

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freenet/river](https://github.com/freenet/river) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
