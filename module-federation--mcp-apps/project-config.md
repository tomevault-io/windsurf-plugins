---
trigger: always_on
description: MCP server that dynamically loads and renders Module Federation components with CORS proxy support.
---

# Module Federation MCP Server

MCP server that dynamically loads and renders Module Federation components with CORS proxy support.

Published as a single package: **`@module-federation/mcp-apps`**

Exports:
- `.` — MCP server (Node.js)
- `./types` — shared TypeScript types
- `./react` — `<UiToolRenderer>` React component for web-based Agent chat UIs

## Architecture

```
src/index.ts          → Entry point: stdio or HTTP server startup
src/server.ts         → MCP server: loads mcp_apps.json, registers tools + resources
src/mcp-app.tsx       → React widget: Module Federation host + global proxy interceptor
src/react/            → ./react export — UiToolRenderer component (AppBridge wrapper)
rsbuild.config.ts     → Single-file HTML build (~660KB)
```

## Core Flow

1. **Tool Call** → MCP server returns resource with `moduleFederation` metadata
2. **Widget Loads** → React app initializes, installs global proxy
3. **Snapshot Fetch** → (legacy snapshot mode only) pre-loads module metadata from snapshot JSON
4. **MF LoadRemote** → Triggers hundreds of cross-origin requests (JS/CSS/images)
5. **Global Proxy** → Intercepts ALL cross-origin requests, routes through MCP
6. **Component Renders** → Remote component displays in iframe

## Key Design Decisions

### Why Global Proxy Instead of Service Worker?

**Chosen:** Global proxy (hijack `window.fetch` and `document.createElement`)

**Why:**
- ✅ Works immediately, no service worker registration delay
- ✅ Synchronous interception of dynamic script/link tags
- ✅ No HTTPS requirement for local development
- ✅ Full control over request/response transformation

**Trade-off:** All cross-origin requests go through slow MCP serial chain

### Why Serial Loading (Not Parallel)?

**Current:** Each resource waits for MCP response before requesting next

**Why:**
- Module Federation's `loadRemote` triggers cascading dependencies
- No way to predict upfront which resources will be needed
- Browser naturally requests dependencies as they're discovered in JS

**Impact:** Major performance bottleneck
- Each MCP call: ~500ms (Front-end → Codex → MCP Server → HTTP → Remote)
- 20 resources = 20 × 500ms = **10 seconds total**

### Legacy Snapshot Mode (`manifestType: 'snapshot'`)

Snapshot mode pre-loads all module metadata before `loadRemote` is called. This is a legacy manifest format used by some CDN providers. New deployments should use `manifestType: 'mf'` (standard Module Federation manifest) instead.

**Benefits:**
- Eliminates manifest.json fetch during `loadRemote`
- Faster module resolution (metadata already in memory)
- Caching across multiple component loads

**Implementation:**
- Snapshot JSON (~100KB) is fetched and injected into a global namespace before creating the MF instance
- Cached in React ref to avoid re-fetch

### Component Extraction Abandoned

**Tried:** Extract LoadingSpinner, ErrorDisplay to separate files

**Failed:** `vite-plugin-singlefile` can't resolve external component imports (644KB → 659KB mismatch, missing code)

**Solution:** Keep all code inline in `mcp-app.tsx`

## Performance Analysis

### Loading Stages (from logs)

```
Step 1: Init snapshot globals (legacy only)  →  <5ms
Step 2: Fetch Snapshot (legacy only)          →  500-1000ms
Step 3: Create MF Instance                    →  10-50ms
Step 4: loadRemote + Resources                →  8000-15000ms  ⚠️ BOTTLENECK
Total:                                            10-16 seconds
```

### Step 4 Breakdown

`loadRemote` triggers:
- 1× manifest.json (if no snapshot)
- 5-10× JS files (main bundle + chunks)
- 10-20× CSS files (component styles)
- 0-10× Images/fonts

**Each resource:**
- ~500ms average MCP round-trip
- Includes: network latency + MCP protocol overhead + HTTP request

**Why so slow:**
- Serial execution (no parallel MCP calls)
- Long proxy chain: Widget → Codex Desktop → MCP Server → HTTP → CDN
- No local cache between requests

### Resource Loading Logs (Enhanced)

Now shows:
```
🌐 [1] JS: main.abc123.js
✅ [1] JS 145KB - 523ms

🌐 [2] CSS: styles.def456.css  
✅ [2] CSS 23KB - 487ms

📊 Proxied 2 resources in 1.0s...
```

Final report:
```
🎯 Load complete - total 12.3s
📊 Performance:
  • Snapshot fetch: 687ms
  • loadRemote: 11234ms ⚠️ main bottleneck
📦 Cross-origin resources: 18
⏱️ Average per resource: 624ms
```

## Tools

### `fetch_remote_resource` (internal)
Proxies cross-origin HTTP requests. Called by widget's global proxy interceptor.

**Input:**
```json
{
  "url": "https://cdn.example.com/app.js",
  "method": "GET"
}
```

**Output:**
```json
{
  "status": 200,
  "body": "...",
  "headers": { "content-type": "application/javascript" }
}
```

## Known Issues

### 1. Loading Too Slow (10-15 seconds)

**Root Cause:** Serial MCP proxy calls for every resource

**Workarounds:**
- Use CDN with low latency
- Pre-cache snapshot (already implemented)
- Reduce total resource count (bundle/compress assets)

**Future Optimizations:**
- Batch MCP requests (load 5-10 resources in parallel)
- Add local cache layer (localStorage/IndexedDB)
- Implement resource prefetch based on manifest

### 2. Protocol-Relative URLs (`//cdn.example.com`)

Some snapshots contain `//` URLs which fail in proxy context.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [module-federation/mcp-apps](https://github.com/module-federation/mcp-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
