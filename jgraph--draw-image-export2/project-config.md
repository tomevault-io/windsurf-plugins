---
trigger: always_on
description: Server-side PNG/JPG/PDF export service for draw.io diagrams using Node.js, Puppeteer, and headless Chrome.
---

# draw-image-export2

Server-side PNG/JPG/PDF export service for draw.io diagrams using Node.js, Puppeteer, and headless Chrome.

## Architecture

This is a **single-file stateless HTTP microservice** (`export.js`, ~784 lines). It accepts diagram XML via HTTP, renders it using draw.io's own rendering engine in headless Chrome, and returns an image or PDF.

```
HTTP Request (xml/xmldata + format + params)
  → Express server (clustered, one worker per CPU)
    → Puppeteer launches headless Chrome per request
      → Navigates to draw.io's export3.html
      → Calls render() with diagram data
      → Waits for #LoadingComplete selector
    → Captures screenshot (PNG/JPG) or PDF
    → Post-processes output (embed XML, DPI, PDF compression)
  → HTTP Response (image/pdf/base64)
```

### Key design decisions
- **Rendering is delegated to draw.io** — the service navigates Puppeteer to `viewer.diagrams.net/export3.html` (configurable via `DRAWIO_BASE_URL`) and lets draw.io's JS engine do all diagram rendering
- **One browser instance per request** — simple but expensive; comment in code notes future optimization to reuse pages
- **Node.js cluster mode** — master forks one worker per CPU core (round-robin scheduling), auto-restarts dead workers
- **30-second timeout** — force-closes browser to prevent zombie Puppeteer processes
- **No authentication in code** — `API_TOKENS` env var is defined in `app.json` but not enforced in `export.js`
- **CORS fully open** — `Access-Control-Allow-Origin: *` on all responses

## Project structure

```
export.js              # Entire application — server, routing, rendering, PNG/PDF post-processing
package.json           # Dependencies and scripts
Procfile               # Heroku process definition
app.json               # Heroku app manifest (buildpacks, env vars)
LICENSE                # Apache 2.0
README.md              # API documentation and usage
iisnode/
  web.config           # Windows IIS deployment config
  README.md            # IIS deployment guide
```

## Key sections in export.js

| Lines | Section | Purpose |
|-------|---------|---------|
| 1-13 | Imports | express, puppeteer, pdf-lib, jsdom, zlib, crc, winston, etc. |
| 14-43 | Cluster setup | Master process forks workers; restarts on death |
| 46-83 | `minimal_args` | Extensive Chrome launch flags for headless server use |
| 85-127 | Express & logging | App setup, body parsing (10MB limit), compression, morgan, winston |
| 129-260 | `writePngWithText()` | Low-level binary PNG manipulation — injects tEXt/zTXt/pHYs chunks before IDAT |
| 262-263 | Routes | `GET /{*splat}` and `POST /{*splat}` → `handleRequest` |
| 265-771 | `handleRequest()` | Core request handler — XML extraction, Puppeteer rendering, format-specific output |
| 773-783 | Server listen | Starts Express on PORT |

### handleRequest flow
1. **Parameter merge** — combines `req.body`, `req.params`, `req.query`
2. **XML extraction** — supports `xmldata` (compressed), `xml` (raw), HTML doc (JSDOM extracts `mxgraph` div), SVG (extracts `content` attribute)
3. **Validation** — requires `format`, `xml`, and `w * h <= MAX_AREA` (20000x20000)
4. **Puppeteer render** — launches browser, navigates to export3.html, calls `render(arg)`, waits for `#LoadingComplete`
5. **Output** — PNG/JPG: screenshot + optional DPI/XML embedding; PDF: `page.pdf()` + pdf-lib compression + optional XML in Subject metadata

### writePngWithText
Manually parses PNG binary format to insert metadata chunks before IDAT:
- `tEXt` — uncompressed key/value (used for embed data)
- `zTXt` — zlib-compressed key/value (used for `mxGraphModel` XML embedding)
- `pHYs` — pixel density / DPI metadata
- Recalculates CRC checksums using `crc.crcjam`

## Commands

```bash
npm install          # Install dependencies (includes Puppeteer which downloads Chrome)
npm start            # Start production server (node export.js)
npm run devstart     # Start with nodemon (auto-reload on file changes)
```

The server listens on `PORT` env var (default: `8000`).

## Environment variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `PORT` | HTTP listen port | `8000` |
| `NO_CLUSTER` | Set to `1` to disable cluster mode (single process) | unset (clustering enabled) |
| `WORKER_POOL_SIZE` | Override number of cluster workers | CPU count |
| `NODE_ENV` | Set to `production` to suppress console logging | unset |
| `DRAWIO_BASE_URL` | Override draw.io rendering host URL | `https://viewer.diagrams.net` |
| `DRAWIO_SERVER_URL` | Legacy alias for `DRAWIO_BASE_URL` | unset |
| `ALLOW_HTTP` | Allow insecure HTTP requests (Heroku) | `false` |
| `API_TOKENS` | Comma-separated API keys for `x-api-key` header (Heroku, not enforced in code) | unset |

## Dependencies

| Package | Purpose |
|---------|---------|
| `express` ^5.1.0 | HTTP server (Express v5) |
| `puppeteer` ^24.8.2 | Headless Chrome for rendering |
| `pdf-lib` ^1.17.1 | PDF post-processing (compression, XML embedding in Subject metadata) |
| `jsdom` ^26.1.0 | Server-side DOM parsing for XML extraction from HTML/SVG |
| `compression` ^1.8.0 | gzip response compression |
| `morgan` ^1.9.1 | HTTP request logging (Apache combined format) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jgraph/draw-image-export2](https://github.com/jgraph/draw-image-export2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
