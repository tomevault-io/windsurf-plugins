---
trigger: always_on
description: This document provides a comprehensive reference for LLMs (Claude, GPT, etc.) working with the Irgo framework.
---

# Irgo Framework - LLM Reference

This document provides a comprehensive reference for LLMs (Claude, GPT, etc.) working with the Irgo framework.

## Framework Overview

Irgo is a **hypermedia-driven application framework** for building cross-platform apps (iOS, Android, desktop, web) using Go + Datastar + Templ. It follows the hypermedia architecture where the server returns HTML fragments via SSE (Server-Sent Events), not JSON.

### Core Concept

```
User Interaction → Datastar Request → Go Handler → Templ Template → SSE Response → DOM Update
```

### Datastar Overview

[Datastar](https://data-star.dev) is a lightweight (~11KB) hypermedia framework that uses:
- **SSE (Server-Sent Events)** for server responses
- **Reactive signals** for client-side state
- **`data-*` attributes** for declarative behavior

### Platform Modes

| Mode | Architecture | Entry Point | Build Tag |
|------|-------------|-------------|-----------|
| **Mobile** | Virtual HTTP via gomobile bridge | `main.go` | `!desktop` |
| **Desktop** | Real HTTP server + native webview | `main_desktop.go` | `desktop` |
| **Web/Dev** | Real HTTP server + browser | `main.go` (serve mode) | `!desktop` |

## Project Structure

```
myapp/
├── main.go              # Mobile/web entry (//go:build !desktop)
├── main_desktop.go      # Desktop entry (//go:build desktop)
├── go.mod
├── app/
│   └── app.go           # Router setup
├── handlers/
│   └── handlers.go      # HTTP handlers
├── templates/
│   ├── layout.templ     # Base HTML layout
│   └── *.templ          # Page/component templates
├── static/
│   ├── css/output.css   # Tailwind CSS
│   └── js/datastar.js   # Datastar library
└── mobile/
    └── mobile.go        # Mobile bridge (optional)
```

## Key Packages

### `github.com/stukennedy/irgo/pkg/router`

Chi-based router with Datastar SSE conveniences.

```go
import "github.com/stukennedy/irgo/pkg/router"

r := router.New()

// Standard handlers return (string, error) for HTML responses
r.GET("/path", func(ctx *router.Context) (string, error) {
    return "<div>HTML</div>", nil
})

// Datastar SSE handlers return error only, use ctx.SSE() for responses
r.DSGet("/path", func(ctx *router.Context) error {
    sse := ctx.SSE()
    return sse.PatchTempl(templates.MyComponent())
})

r.DSPost("/path", handler)
r.DSPut("/path", handler)
r.DSPatch("/path", handler)
r.DSDelete("/path", handler)

// URL parameters
r.DSGet("/users/{id}", func(ctx *router.Context) error {
    id := ctx.Param("id")
    // ...
    return nil
})

// Route groups
r.Route("/api", func(r *router.Router) {
    r.DSGet("/users", listUsers)
})

// Static files
r.Static("/static", http.Dir("static"))

// Get the http.Handler
handler := r.Handler()
```

### `router.Context` - Request/Response Helpers

```go
// Standard handler (returns HTML string)
func handler(ctx *router.Context) (string, error) {
    // Input
    ctx.Param("id")           // URL path parameter
    ctx.Query("q")            // Query string parameter
    ctx.FormValue("name")     // Form field value
    ctx.Header("X-Custom")    // Request header

    // Datastar detection
    ctx.IsDatastar()          // true if Accept: text/event-stream

    // Output - HTML responses (for full page loads)
    ctx.HTML("<div>content</div>")
    ctx.HTMLStatus(201, "<div>created</div>")

    // Output - JSON responses
    ctx.JSON(data)
    ctx.JSONStatus(201, data)

    // Output - Errors
    ctx.Error(err)
    ctx.ErrorStatus(500, "message")
    ctx.NotFound("not found")
    ctx.BadRequest("invalid input")

    // Output - Redirects
    ctx.Redirect("/new-url")

    // Output - No content
    ctx.NoContent()

    return "<div>response</div>", nil
}

// Datastar SSE handler (returns error only)
func sseHandler(ctx *router.Context) error {
    // Read signals from request body
    var signals struct {
        Name string `json:"name"`
        Page int    `json:"page"`
    }
    ctx.ReadSignals(&signals)

    // Get SSE writer
    sse := ctx.SSE()

    // Patch HTML into DOM (morphs elements by ID)
    sse.PatchTempl(templates.MyComponent(data))
    sse.PatchHTML(`<div id="result">Updated</div>`)

    // Update client-side signals
    sse.PatchSignals(map[string]any{"count": 5})

    // Remove elements
    sse.Remove("#old-element")

    // Redirect browser
    sse.Redirect("/new-url")

    return nil
}
```

### `github.com/stukennedy/irgo/pkg/datastar`

SSE wrapper for Datastar responses with templ integration.

```go
import "github.com/stukennedy/irgo/pkg/datastar"

// Create SSE writer manually (usually use ctx.SSE() instead)
sse := datastar.NewSSE(w, r)

// Patch operations (morph DOM elements)
sse.PatchTempl(templates.Component())           // Render templ and patch
sse.PatchHTML(`<div id="x">HTML</div>`)         // Patch raw HTML

// With options
sse.PatchTempl(comp, datastar.WithModeOuter)    // Replace entire element
sse.PatchTempl(comp, datastar.WithModeAppend)   // Append to element

// Update client signals
sse.PatchSignals(map[string]any{
    "count": 10,
    "name": "John",
})

// Remove elements by selector
sse.Remove("#element-id")
sse.Remove(".class-name")

// Browser navigation
sse.Redirect("/new-page")

// Read signals from request
var signals MyStruct

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stukennedy/irgo](https://github.com/stukennedy/irgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
