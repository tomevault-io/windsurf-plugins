---
trigger: always_on
description: Turns raw app screenshots into store-ready App Store and Google Play assets, in
---

# AppEditions

Turns raw app screenshots into store-ready App Store and Google Play assets, in
every language a project ships in. Go + [howl-go](../howl-go) + templ +
shadcn-templ, in an OS-native window. No Node at run time, no Electron, no
canvas in the browser.

Read [`docs/howl-go.llms.txt`](docs/howl-go.llms.txt) before writing framework
code and [`docs/shadcn-templ.md`](docs/shadcn-templ.md) before touching the UI.
Neither set of conventions is guessable.

## The one decision everything rests on

**There is one renderer, and it runs on the server.**

`render.Scene(w, h, screen, settings, copy, sources)` draws a tile. The editor's
preview asks for a 240px tile and gets a PNG; the export asks for 1320×2868 and
gets a PNG. Same function, same call, only `w`/`h` differ.

The app this replaces drew its preview on a `<canvas>` in the page and its
export on the same canvas at full size, which was already the right idea. Moving
it to the server removes the browser from the question entirely: the preview is
not a faithful reimplementation of the export, it *is* the export at a smaller
size. Every tile in the interface is an `<img>` pointed at
`/api/preview/{project}/{screen}`.

Consequences worth knowing before changing anything:

- There is no client-side model of a project. Controls post to an endpoint and
  the page re-renders. `client/public/forge.js` is the whole client, and it
  binds by delegation on `document` because a local navigation replaces
  `#outlet` wholesale.
- Rendering is CPU-bound Go. A preview is ~15 ms, a full-size tile ~300 ms. The
  export fans out one goroutine per tile, bounded to one per core.
- A preview response carries an ETag over everything the drawing depends on, so
  the editor's reload-everything-after-a-change is mostly 304s.

## Build

```bash
make                 # fsapis -> fsroutes -> templ generate -> go build
make run             # the app, in its own window
make dev             # the same, watched: rebuild, restart, reload in place
make serve           # just the server on :9010, for a browser
make dev-web         # the watched loop without a window
make test
make check           # howl check — the conventions, enforced
make package         # dist/AppEditions.app (or the Linux install tree)
make css             # ONLY when you used a Tailwind class no source used before
```

`make dev` is the loop: `howl dev` rebuilds and restarts the server on save
while keeping its proxy port up, and the window attaches to that port once and
reloads its content in place. The window must not be `howl dev`'s child — dev
restarts what it builds, and that would kill and respawn the window on every
save.

`make css` is the only target that needs Node. `client/public/app.css` is a
committed build artifact; run `make css` **before** `make`, never after — the
binary embeds `client/public`, so a stylesheet rebuilt after the go build is one
the server does not serve.

## Layout

```
main.go                     flags, then boot
boot/boot.go                app.New(...) — the same app for the server and the window
desktop/                    the native window; nested module, cgo, WKWebView
internal/model/             the vocabulary: project, screen, settings, template. A leaf.
internal/presets/           the tables: devices, layouts, arrangements, rhythms, sizes, locales
internal/render/            THE renderer: scene, text, frames, shadow, fonts
internal/store/             SQLite documents + the screenshot directory
internal/exporter/          full-size render to disk, and the preview
internal/ai/                claude / codex, for writing and translating copy
server/apis/                typed endpoints — file location is the URL (fsapis)
server/handlers/            the page loader, the PNG renderer, the multipart upload
client/pages/               the routes (fsroutes)
client/ui/                  shared components + the icon registry
client/view/                what a page is allowed to see
client/styles/app.css       stylesheet source -> client/public/app.css
```

## Invariants

Each of these exists because breaking it produced a real bug.

1. **Resolve overrides in exactly one place** — the top of `render.Scene`.
   Callers pass the raw screen and the project settings. Two places resolving
   inheritance would eventually disagree, and the disagreement would be
   invisible until an exported PNG came out different from the preview it was
   approved from.

2. **An override of `nil` means inherit.** Never write a resolved value into
   `Overrides` to mean "same as the set" — that silently pins it and the screen
   stops following the global for the rest of its life. `apistore.Setting.Pin`
   is the only thing that writes one.

3. **A `font.Face` is not safe for concurrent use.** `opentype.Face` keeps one
   glyph buffer and one rasteriser. Two goroutines drawing through the same face
   panic inside `image/draw` with an index out of range — and only once the
   export renders tiles in parallel. `render.Face` hands out faces behind a
   mutex; keep it that way.

4. **Fonts are static TTFs, not variable.** `x/image/font/sfnt` reads a variable
   font's default instance and cannot select an axis, so a variable Inter would

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirairoad/appeditions](https://github.com/mirairoad/appeditions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
