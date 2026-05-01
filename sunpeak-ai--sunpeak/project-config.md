---
trigger: always_on
description: Note that "sunpeak", except where required in URLs or code, is always lowercase.
---

# sunpeak

Note that "sunpeak", except where required in URLs or code, is always lowercase.

sunpeak is a framework for building MCP Apps with interactive UIs that run inside AI chat hosts (ChatGPT, Claude, and future major hosts). Built on top of the MCP Apps SDK (`@modelcontextprotocol/ext-apps`).

The value proposition of the sunpeak framework is to help developers and their agents:

1. Test MCP Apps locally and automatically (in CI/CD) using a replica of the ChatGPT and Claude runtimes.
   1. Save time manually testing all possible host, server, app, ui, and backend states.
   2. Protect developers from 4-click manual refreshes on every code change in each host.
   3. Cancel all the $20 per person per host per month testing accounts.
   4. Avoid burning host credits on every test and code change.
2. Build multi-platform MCP Apps in a structured way that's easy to understand and get started.
3. Test their MCPs in ChatGPT with HMR and Claude with automatic rebuilds and refresh notifications.

## Quick Reference

```bash
pnpm --filter sunpeak test -- --run    # Unit tests (vitest)
pnpm --filter sunpeak lint             # ESLint
pnpm --filter sunpeak typecheck        # tsc --noEmit
pnpm --filter sunpeak build            # Vite build
pnpm --filter sunpeak validate         # Full validation (lint + build + test + examples)
pnpm --filter sunpeak generate-examples  # Regenerate examples/ from template
```

## Architecture

**All resource content renders inside iframes** — never directly in the host page. This matches how AI chat hosts (ChatGPT, Claude) display apps and enables direct re-export of SDK hooks.

### Multi-Host Inspector

The inspector supports multiple host platforms via a **HostShell** abstraction. Each host provides:
- **Conversation chrome** — the visual shell (message bubbles, headers, input areas)
- **Theme** — host-specific CSS variables and theme application
- **Host info & capabilities** — reported to the app via MCP protocol

Switching hosts in the sidebar changes the conversation chrome, theming, and reported host info/capabilities. The sidebar controls, iframe infrastructure, and state management are shared.

### Rendering Flow (Double-Iframe Sandbox Architecture)
1. `Inspector` (host page) → `HostShell.Conversation` → `IframeResource`
2. `IframeResource` creates an outer `<iframe>` containing a **sandbox proxy** that relays PostMessage between the host and an inner iframe holding the actual app. This two-level architecture matches how production hosts (ChatGPT, Claude) isolate app iframes on a separate origin (e.g., `web-sandbox.oaiusercontent.com`).
   - **Outer iframe**: Loads the sandbox proxy from a separate-origin server (port 24680) or via `srcdoc` (fallback for unit tests).
   - **Inner iframe**: Created by the proxy, loads the app HTML via `src` (dev: Vite HMR URL) or `document.write()` (prod: generated HTML).
3. `McpAppHost` wraps the SDK's `AppBridge` for host-side communication. Messages flow: host ↔ outer iframe (proxy) ↔ inner iframe (app), all via PostMessage relay.
4. Inside the inner iframe, the resource component uses `useApp()` which connects via `PostMessageTransport` to `window.parent` (the proxy), which relays to the host.

**The cross-origin relationship between iframes is intentional and must be preserved.** The outer iframe (sandbox proxy, port 24680), inner iframe (resource HTML, proxied through the inspector on port 3000), and the Vite dev server (port 8000) are deliberately on different origins. This replicates production CSP behavior where the host, sandbox, and app content live on separate origins. Do not "fix" cross-origin issues by collapsing these onto the same origin — that would make the inspector less representative of production and mask real CSP bugs.

**Safari is incompatible with `sunpeak dev`.** Safari upgrades cross-origin HTTP requests to HTTPS, which breaks the multi-port localhost architecture. This is a known limitation with no workaround. Use Chrome for development. Production deploys (`sunpeak start`) work in all browsers because the app is a single bundled page without cross-origin localhost dependencies.

### E2E Tests
Tests use `page.frameLocator('iframe').frameLocator('iframe')` to access resource content inside the double-iframe. Elements on the inspector chrome (header, `#root`) use `page.locator()` directly. Console error tests filter expected MCP handshake errors.

### Live Tests (`pnpm test:live`)
Automated tests against real ChatGPT using Playwright. Uses the same `ChatGPTPage` class for selectors, message sending, and iframe handling. Auth flow: saved session → manual login in the opened browser window. Sessions typically last only a few hours because Cloudflare's HttpOnly `cf_clearance` cookie cannot be persisted by `storageState()`. The `global-setup.mjs` handles auth + MCP server refresh in the same browser session (refresh must happen before the browser closes while `cf_clearance` is still valid).

## Package Structure

```
packages/sunpeak/
├── src/
│   ├── index.ts              # Main barrel: SDK re-exports + hooks + types
│   ├── inspector/            # Generic multi-host inspector core
│   │   ├── inspector.tsx     # Inspector component (host picker, sidebar, delegates to shell)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sunpeak-AI/sunpeak](https://github.com/Sunpeak-AI/sunpeak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
