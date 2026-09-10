---
trigger: always_on
description: A concise, code-accurate guide to use, extend, and safely modify xun without reading all source files.
---

# Xun Development Guide for AI Agents

A concise, code-accurate guide to use, extend, and safely modify xun without reading all source files.

## 1) Core concepts
- App: main instance holding ServeMux, routes, global middlewares, view engines, viewers, compressors, template func map, and asset URL map.
- Router/Group: register routes and middlewares. Uses Go 1.22 ServeMux patterns: "METHOD pattern".
- Middleware: func(next HandleFunc) HandleFunc.
- Context: wraps Request/Response, Routing, App, TempData. Provides View, Redirect, Accept, AcceptLanguage, Get/Set.
- Viewer: renders response by content negotiation (HTML/JSON/Text/XML/File/String).
- ViewEngine: loads templates/static files from fs.FS; supports hot-reload in dev.
- ResponseWriter: wraps http.ResponseWriter; tracks status/body bytes; supports gzip/deflate transparently.

## 2) App creation and lifecycle
Canonical six-step `main()` shape: load config → open resources → build app → register routes → start app → run listeners.

```go
err := loadConfig()
db, err := setupDB(ctx)

mux := http.NewServeMux()
app := xun.New(
    xun.WithMux(mux),                              // always pass your own mux; default pollutes http.DefaultServeMux
    xun.WithFsys(getFsys()),                       // dev: os.DirFS("./app"); prod: //go:embed sub
    xun.WithWatch(),                               // DEV ONLY — thread-unsafe
    xun.WithHandlerViewers(&xun.JsonViewer{}),     // content negotiation
    xun.WithInterceptor(htmx.New()),               // cross-cutting wrappers
    xun.WithBuildAssetURL(func(p string) bool {
        return strings.HasPrefix(p, "/assets/")
    }),
    xun.WithCompressor(&xun.GzipCompressor{}, &xun.DeflateCompressor{}),
)

setupRoutes(app)        // xun-managed (go through middleware pipeline)
app.Start()             // non-blocking — registers handlers on mux
defer app.Close()
http.ListenAndServe(":80", mux)
```

Notes:
- `WithWatch` is not thread-safe; enable only in development. In production embed assets and disable Watch.
- Always pass `WithMux(http.NewServeMux())` to `xun.New`. Skipping this lets `app.Mux()` fall back to `http.DefaultServeMux` and pollutes global state.

#### Dev / prod twin fs.FS
```go
//go:embed app
var fsys embed.FS
func getFsys() fs.FS {
    if fi, err := os.Stat("./app"); err == nil && fi.IsDir() {
        return os.DirFS("./app")       // dev: live files
    }
    sub, _ := fs.Sub(fsys, "app")      // prod: bundled binary
    return sub
}
```

## 3) Project structure and file routing
- public: static assets -> GET /...; public/index.html -> GET /{$}.
- components: reusable html fragments.
- layouts: page layouts; choose via <!--layout:name--> at top of page.
- pages: filesystem-based page routing; pages/foo/index.html -> GET /foo/{$}.
- views: internal views (not auto-routed), referenced by Context.View with viewer name.
- text: text templates (text/template); MIME/charset auto-detected from filename/content.
- Dynamic segments: {var} in file/dir names, e.g. pages/user/{id}.html -> GET /user/{id}.
- Multiple hosts: top-level folder like pages/@abc.com/index.html -> GET abc.com/{$}.

## 4) Routing and handlers
- Handler signature: func(c *xun.Context) error
```go
app.Get("/users/{id}", func(c *xun.Context) error {
  id := c.Request.PathValue("id")
  return c.View(User{Name: id})
})
```
- Groups and middlewares:
```go
admin := app.Group("/admin")
admin.Use(authMiddleware)
admin.Get("/{$}", handler)
```
- Error contract:
  - return nil: response done.
  - return xun.ErrCancelled: stop chain (you already handled response/redirect/status).
  - return other error: framework writes 500 + X-Log-Id; if xun.ErrViewNotFound then 404.

#### PageRoute with path parameters
A page file with a `{var}` segment auto-registers a PageRoute with the matching Go 1.22 pattern. To supply the model, override the same pattern with `app.Get` (or `g.Get` on a group):

```go
// pages/user/{id}.html    →  auto-registers GET /user/{id}
app.Get("/user/{id}", func(c *xun.Context) error {
    id, _ := strconv.ParseInt(c.Request.PathValue("id"), 10, 64)
    var u User
    db.QueryRowContext(ctx, "SELECT id, name, email FROM users WHERE id = ?", id).
        Scan(&u.ID, &u.Name, &u.Email)
    return c.View(u)        // ← .Data = u
})
```

> **Path syntax:** only `{id}` (Go 1.22 brace form) works. `:id` colon-style is NOT understood by xun.

#### The `{$}` root pattern
`pages/index.html` auto-registers `GET /{$}` (xun appends `{$}` to patterns ending in `/` so they only match with the trailing slash). To supply data to the root page you must register against the same pattern:

```go
// Right — matches the auto-registered PageRoute
app.Get("/{$}", handleLanding)

// Wrong — different pattern, will not be called for /
// app.Get("/", handleLanding)
```

The same rule applies to `pages/<group>/index.html` inside `app.Group(...)`.

## 5) Middleware order
- App.Use applies globally; Group.Use applies to that group only.
- Construction is inside-out; execution is outer to inner; handler last.

## 6) Content negotiation and Viewers
- A route can have multiple viewers; chosen by Accept header. If none matches, fallback to the first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaitoo/xun](https://github.com/yaitoo/xun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
