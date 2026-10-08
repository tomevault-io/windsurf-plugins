---
trigger: always_on
description: Lightweight macOS browser built on wry/tao. Performance-first, minimal footprint. AI assistant via ACP (Agent Client Protocol). MCP server for external AI tool control.
---

# Octoweb Developer Guide

Lightweight macOS browser built on wry/tao. Performance-first, minimal footprint. AI assistant via ACP (Agent Client Protocol). MCP server for external AI tool control.

## Architecture

```
main.rs                      ← App entry, event loop, WebView pool, keyboard (CGEventTap)
   │
   ├── browser.rs            ← TabManager (tabs, history, active state, visit counts)
   ├── config.rs             ← Config, session persistence, favicon cache, prompt history
   ├── acp.rs                ← ACP integration (sidebar AI + background learning agent)
   ├── mcp.rs                ← MCP server — 31 browser control tools over HTTP JSON-RPC
   ├── agent_tabs.rs         ← MCP-driven tabs: kept rendering off-screen, exempt from hibernation
   │
   ├── url.rs                ← URL resolution (user input → navigable URL)
   ├── webview_utils.rs      ← Injected JS scripts, favicon cache lookup, overlay data
   ├── macos.rs              ← macOS-specific: dock icon, Edit menu, MRU list, external URLs
   ├── nav_error_patch.rs    ← WKWebView navigation error callbacks (ObjC runtime patch)
   ├── content_rules.rs      ← WKContentRuleList (ad/tracker blocking)
   ├── crash_report.rs       ← WebContent process termination tracking
   ├── cold_open.rs          ← First-run setup, URL scheme handler (kAEGetURL)
   ├── hibernation.rs        ← Session save/restore (tabs, URLs)
   ├── tab_stats.rs          ← Per-tab CPU/RSS stats from WebContent XPC process
   ├── quickslots.rs         ← Fast-access slot logic (⌘⇧1–0 save, ⌘1–0 open)
   ├── prompt_history_js.rs  ← Shared JS module: prompt history nav, Ctrl+R search, ghost text
   │
   ├── page_index.rs         ← Full-text index of visited page text (palette `/query`, MCP search tool)
   ├── later.rs              ← Save-for-later queue (⌘⇧L) — decays after later_days, per workspace
   ├── markdown_js.rs        ← Page → markdown walker for ⌥⌘C
   ├── overlay_html.rs       ← Command palette (⌘⇧P) — fuzzy search tabs/history, `/` searches page text
   ├── sidebar_html.rs       ← AI assistant sidebar (⌘⇧A) — chat UI
   ├── address_bar_html.rs   ← Address bar with URL display + AI button
   ├── inline_edit_html.rs   ← Inline AI edit modal (⌘⇧E) — text transformation
   ├── settings_html.rs      ← Settings modal (⌘,)
   ├── shortcuts_html.rs     ← Keyboard shortcuts overlay (⌘/)
   ├── newtab_html.rs        ← New tab page (quick slots, search)
   ├── find_bar_html.rs      ← Find-in-page bar (⌘F)
   ├── quickslots_html.rs    ← Quick slots footer bar UI
   ├── notification_html.rs  ← Toast notification UI
   ├── error_page_html.rs    ← Custom error page for load failures
   └── progress_bar_html.rs  ← Top-of-screen page load progress bar
```

## Key Subsystems

### ACP — AI Assistant & Background Learning (`acp.rs`)

Two ACP connections run independently:

1. **Sidebar assistant** (`octoweb:assistant`) — interactive chat, user-initiated
2. **Background learner** (`octoweb:learning`) — periodic, silent, memorizes browsing patterns

**Architecture:**
- Each `AcpHandle` spawns a dedicated OS thread with its own `tokio::runtime` + `LocalSet`
- Subprocess: `octomind acp <agent_tag>` with piped stdin/stdout
- Communication: `mpsc` channels (events → main thread, prompts → agent thread)
- Main thread polls via `AcpHandle::poll()` on every event loop tick
- Process killed on drop (`kill_on_drop(true)`)

**Idle timeout:** Resets on every `session_notification` — only fires after 5 min of silence (not 5 min total). Safe for long tool chains.

**Reconnection:** Exponential backoff (1s→2s→4s→8s→16s→30s), max 5 retries. Generation counter invalidates stale timers.

### MCP — Browser Control Server (`mcp.rs`)

HTTP JSON-RPC on `localhost:3434/mcp` (`OCTOWEB_MCP_PORT` overrides; `OCTOWEB_CONFIG_DIR` isolates the profile for side-by-side instances). External AI agents control the browser via 31 tools:

| Category | Tools |
|----------|-------|
| Navigation | `browser_navigate` (always background), `browser_go_back`, `browser_go_forward`, `browser_reload`, `browser_wait` |
| Tab management | `browser_get_tabs`, `browser_get_current_tab`, `browser_switch_tab`, `browser_close_tab` |
| Page content | `browser_snapshot`, `browser_get_page_info`, `browser_get_page_content`, `browser_execute_js`, `browser_screenshot` |
| Interaction | `browser_click`, `browser_hover`, `browser_type`, `browser_scroll`, `browser_press_key`, `browser_select_option` |
| Dialogs & uploads | `browser_handle_dialog`, `browser_upload_file` (arm before the triggering click) |
| Diagnostics | `browser_console_messages`, `browser_network_requests` |
| History & media | `browser_get_history`, `browser_search_history_content`, `browser_get_later`, `browser_get_playing_tabs` |

Interactions run through an actionability harness (`dom_actions.rs`): resolve with retry, bounding-box stability, occlusion detection, pointer+mouse event sequences. Async scripts evaluate via `callAsyncJavaScript` (`async_eval.rs`) so Promises are actually awaited. End-to-end suite: `test_mcp.py` + `tests/fixtures/` (28 tests against a live instance).

**Architecture:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Muvon/octoweb](https://github.com/Muvon/octoweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
