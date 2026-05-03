---
trigger: always_on
description: This is an R package that provides a lightweight interface to the AntV G2
---

# gglite - Repository Instructions for Copilot

## Repository Overview

This is an R package that provides a lightweight interface to the AntV G2
JavaScript visualization library with a ggplot2-style API. It supports rendering
in R Markdown, litedown, Shiny, and standalone HTML previews.

**Project Type**: R package **Languages**: R, JavaScript (via CDN) **Size**:
Small repository (\~15 R source files)

## Build and Test Instructions

### Prerequisites

R has been automatically installed via `.github/copilot-setup-steps.yml` when
working with GitHub Copilot.

### Bootstrap and Build Sequence

1.  **Build the R package**:

    ``` bash
    R CMD build .
    ```

2.  **Install the package**:

    ``` bash
    R CMD INSTALL *_*.tar.gz
    ```

### Testing Conventions

-   Tests are in `tests/testit/test-*.R`
-   Use `testit` package for assertions
-   Always wrap test conditions in `{}`: `assert('message', {})`
-   Use `has_error()` instead of `tryCatch()` for error testing
-   Load the package with `library(gglite)` before testing
-   Use `%==%` (from testit) instead of `==` to test for strict equality
-   Never use `:::` to access internal functions in tests; testit exposes
    internal functions automatically, so call them directly

### Rendering Examples to HTML

All `examples/*.Rmd` files are rendered using **litedown**, not rmarkdown. Never
use `rmarkdown::render()` — it will produce incorrect output or fail.

To render an Rmd file to HTML (e.g., for inspection or headless browser
testing):

``` bash
Rscript -e 'litedown::fuse("path/to/foo.Rmd")'
# output: path/to/foo.html
```

This will render a self-contained HTML file (since we have enabled the
`embed_resources` option for litedown in `copilot-setup-steps.yml`). All
external JS/CSS resources are embedded, so they won't be blocked when testing
via headless browsers.

Similarly, for an R script that generates a g2 plot, you can render it to HTML
via:

``` bash
Rscript -e 'litedown::fuse("path/to/foo.R')'
# output: path/to/foo.html
```

When you do headless browser testing for an example, you must always write the
example code to an .R file and render it this way.

The GitHub Pages site is built by the `yihui/litedown/site` action, which calls
`litedown::fuse()` for every Rmd in the repo.

### Testing Plots in Headless Browsers

Since gglite generates HTML/JavaScript visualizations, **plots must be tested in
headless browsers** to make sure they can be rendered correctly and produce no
errors in the browser console. The workflow is:

> **Prefer SVG rendering for headless tests.** Set
> `options(gglite.renderer = 'svg')` before calling `litedown::fuse()` so the
> rendered output contains an `<svg>` element that you can inspect directly in
> the DOM. SVG attributes (fill, stroke, etc.) are directly readable without
> relying on canvas pixel values, making theme and style verification much
> easier. Use `document.querySelectorAll('svg rect[fill]')` or similar to check
> theme properties.

1.  **Render to a full HTML page** — set `options(gglite.renderer = 'svg')`,
    then render both `.Rmd` and `.R` files to `.html` via `litedown::fuse()`.
    The output is self-contained (all JS/CSS embedded) because `embed_resources`
    is enabled in `copilot-setup-steps.yml`.

2.  **Open with `google-chrome` under Xvfb** and enable remote debugging. Use
    `google-chrome`, **not** `chromium` — `chromium` crashes in this
    environment. Omit `--no-zygote` and `--single-process`; both cause crashes.
    Use `file://` absolute paths — **Chrome cannot reach `127.0.0.1`** in this
    sandbox so `http://` URLs always fail with a connection error.

    ``` bash
    Xvfb :99 -screen 0 1280x1024x24 2>/dev/null &
    DISPLAY=:99 google-chrome --no-sandbox --disable-gpu \
      --disable-dev-shm-usage \
      --remote-debugging-port=9222 \
      "about:blank" 2>/dev/null &
    sleep 10   # wait for Chrome to start (needs ~10s in this environment)
    ```

    > **Important:** The `playwright-browser_*` tools are sandboxed from the
    > loopback interface. Do **not** use them for local pages. Use the
    > `google-chrome` + CDP approach above instead.

3.  **Query the live DOM via CDP** (Chrome DevTools Protocol) using Python's
    `websockets` package (`pip install websockets`). Always pass
    `max_size=10*1024*1024` to handle large embedded pages:

    ``` python
    import asyncio, json, base64
    import websockets

    WS_URL = (lambda: __import__('json').loads(
        __import__('urllib.request', fromlist=['urlopen'])
        .urlopen('http://127.0.0.1:9222/json').read()
    )[0]['webSocketDebuggerUrl'])()

    rid = [0]

    async def send(ws, method, params=None):
        rid[0] += 1; r = rid[0]
        await ws.send(json.dumps({"id": r, "method": method, "params": params or {}}))
        while True:
            msg = json.loads(await asyncio.wait_for(ws.recv(), timeout=15))
            if msg.get("id") == r: return msg.get("result", {})

    async def js(ws, expr, await_promise=False):
        return await send(ws, "Runtime.evaluate", {
            "expression": expr, "awaitPromise": await_promise, "returnByValue": True
        })

    async def shot(ws, path):
        r = await send(ws, "Page.captureScreenshot", {"format": "png"})

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yihui/gglite](https://github.com/yihui/gglite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
