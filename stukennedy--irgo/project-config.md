---
trigger: always_on
description: This app is built with **Irgo**, a hypermedia-driven framework for cross-platform apps using Go + Datastar + Templ.
---

# Irgo App Development Guide

This app is built with **Irgo**, a hypermedia-driven framework for cross-platform apps using Go + Datastar + Templ.

## Architecture

```
User Interaction → Datastar Request → Go Handler → Templ Template → SSE Response → DOM Update
```

**Key principle:** The server returns HTML fragments via SSE (Server-Sent Events), not JSON. Datastar handles DOM updates.

## Project Structure

```
├── main.go              # Mobile/web entry (//go:build !desktop)
├── main_desktop.go      # Desktop entry (//go:build desktop)
├── app/
│   └── app.go           # Router setup and route definitions
├── handlers/
│   └── handlers.go      # HTTP handlers returning HTML or SSE
├── templates/
│   ├── layout.templ     # Base HTML layout
│   └── *.templ          # Page and component templates
├── static/
│   ├── css/output.css   # Tailwind CSS (generated)
│   └── js/datastar.js   # Datastar library
└── mobile/
    └── mobile.go        # Mobile bridge (optional)
```

## CLI Commands

```bash
irgo dev                 # Web dev server with hot reload
irgo run desktop         # Run as desktop app
irgo run desktop --dev   # Desktop with devtools
irgo run ios --dev       # iOS Simulator
irgo run android --dev   # Android Emulator
irgo templ               # Regenerate templ files
```

## Router & Handlers

### Standard Handlers (Full Page Loads)

Standard handlers return `(string, error)`. The string is HTML.

```go
import (
    "github.com/stukennedy/irgo/pkg/router"
    "github.com/stukennedy/irgo/pkg/render"
)

// Full page load
r.GET("/", func(ctx *router.Context) (string, error) {
    return renderer.Render(templates.HomePage())
})
```

### Datastar SSE Handlers

Datastar handlers return `error` only and use `ctx.SSE()` for responses.

```go
// Datastar SSE endpoint
r.DSGet("/greeting", func(ctx *router.Context) error {
    var signals struct {
        Name string `json:"name"`
    }
    ctx.ReadSignals(&signals)

    sse := ctx.SSE()
    return sse.PatchTempl(templates.Greeting(signals.Name))
})

r.DSPost("/todos", createTodo)
r.DSPut("/todos/{id}", updateTodo)
r.DSPatch("/todos/{id}", toggleTodo)
r.DSDelete("/todos/{id}", deleteTodo)
```

### Context Methods

**Input:**
- `ctx.Param("id")` - URL path parameter
- `ctx.Query("q")` - Query string parameter
- `ctx.FormValue("name")` - Form field value
- `ctx.Header("X-Custom")` - Request header
- `ctx.ReadSignals(&signals)` - Parse Datastar signals from request

**Datastar Detection:**
- `ctx.IsDatastar()` - true if Accept: text/event-stream

**SSE Output (for Datastar handlers):**
```go
sse := ctx.SSE()
sse.PatchTempl(templates.Component())      // Patch templ component
sse.PatchHTML(`<div id="x">HTML</div>`)    // Patch raw HTML
sse.PatchSignals(map[string]any{...})      // Update client signals
sse.Remove("#element-id")                   // Remove element
sse.Redirect("/new-url")                    // Navigate browser
```

**Standard Output (for full page handlers):**
- Return HTML string from handler
- `ctx.Redirect("/path")` - HTTP redirect
- `ctx.NotFound("message")` - 404 response
- `ctx.BadRequest("message")` - 400 response
- `ctx.NoContent()` - 204 response

## Templ Templates

Templ is a type-safe HTML templating language that compiles to Go.

### Basic Syntax

```go
// templates/components.templ
package templates

// Component with parameters
templ UserCard(name string, email string) {
    <div class="card">
        <h2>{ name }</h2>
        <p>{ email }</p>
    </div>
}

// Component with children
templ Card(title string) {
    <div class="card">
        <h3>{ title }</h3>
        { children... }
    </div>
}

// Usage
templ ProfilePage() {
    @Card("Profile") {
        <p>Content goes here</p>
    }
}

// Conditionals
templ Status(active bool) {
    if active {
        <span class="text-green-500">Active</span>
    } else {
        <span class="text-red-500">Inactive</span>
    }
}

// Loops
templ UserList(users []User) {
    <ul>
        for _, user := range users {
            <li>{ user.Name }</li>
        }
    </ul>
}

// Conditional attributes
templ Checkbox(checked bool) {
    <input type="checkbox" checked?={ checked }/>
}

// Dynamic classes
templ Item(done bool) {
    <span class={ "item", templ.KV("line-through", done) }>Item</span>
}

// Safe URLs
templ Link(url string) {
    <a href={ templ.SafeURL(url) }>Link</a>
}

// Raw HTML (use sparingly)
templ RawContent(html string) {
    @templ.Raw(html)
}
```

### Rendering in Handlers

```go
renderer := render.NewTemplRenderer()

// Standard handler
func handler(ctx *router.Context) (string, error) {
    return renderer.Render(templates.MyComponent(data))
}

// Datastar handler
func sseHandler(ctx *router.Context) error {
    sse := ctx.SSE()
    return sse.PatchTempl(templates.MyComponent(data))
}
```

## Datastar Patterns

This project uses **Datastar** from `https://data-star.dev/`. Key concepts:
- **Signals**: Reactive client-side state
- **SSE**: Server responses as event streams
- **`data-*` attributes**: Declarative behavior

### Signals (Client-Side State)

```go
// Initialize signals
templ Counter() {
    <div data-signals="{count: 0}">
        <span data-text="$count">0</span>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stukennedy/irgo](https://github.com/stukennedy/irgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
