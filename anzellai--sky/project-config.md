---
trigger: always_on
description: This is a [Sky](https://github.com/anzellai/sky) project: a pure functional,
---

# AGENTS.md — Sky Language Project

This is a [Sky](https://github.com/anzellai/sky) project: a pure functional,
Elm-family language that compiles to typed Go and ships as a single `sky` binary
(you also need Go 1.21+). **If it compiles, it works** — every side effect returns
`Task Error a`, every fallible value returns `Result Error a`, and `sky check`
runs `go build` on the emitted Go, so shape mismatches surface at check time. No
runtime panics from well-typed code, no nil leakage, no silent numeric coercion.

Type annotations are load-bearing: `f : String -> Int -> Result Error Profile`
rejects a body that would infer wider. Inline records aren't allowed in
signatures — give any record in a signature a `type alias`.

## The full API lives in `sky doc` — not here

This file is orientation only. For **complete, always-current signatures + docs**:

```sh
sky doc --list            # every module (stdlib + your project), incl. Std.Live etc.
sky doc Std.Ui            # one module's exported bindings, with types + summaries
sky doc --serve           # browsable HTML API at http://localhost:8080
```

Reach for `sky doc <Module>` whenever you need an exact signature. Do not
memorise or inline signatures here — they drift; `sky doc` doesn't.

## Choose the app shape first

| You want… | Use | Entry |
|---|---|---|
| Web app (forms, real-time, UI state) | **Sky.Live** | `Live.app (Live.config {…})` |
| HTTP/JSON API (no browser UI) | **Sky.Http.Server** | `Server.listen 8000 [...]` |
| Terminal UI | **Sky.Tui** | `Tui.app (Tui.config {…})` |
| Desktop app (macOS) | **Sky.Webview** | `Webview.app { … }` (closed record) |
| One-shot CLI / cron | **Sky.Cli** | `main = Task.run cmd` |

Before scaffolding more than a proof of concept, confirm with the user:
**persistence** (SQLite default / Postgres for multi-instance / none), **auth**
(`Std.Auth` / OAuth / external), **session store** for Sky.Live (memory dev /
sqlite single-instance / redis|postgres for multi-replica), and **deploy target**.

## Std.Ui is the default for application interfaces

Build UI with **`Std.Ui`** — a typed, no-CSS layout DSL (`row`/`column`/`el` +
typed attributes from `Background`/`Border`/`Font`/`Input`/`Region`). It
HTML-escapes everything and renders identically across Sky.Live, Sky.Tui, and
Sky.Webview. Reach for `Std.Html` only to wrap raw markup. Never write CSS
strings; never emit raw HTML/JS (`data-sky-eval` is forbidden).

```elm
import Std.Ui as Ui
import Std.Ui.Font as Font

view model =
    Ui.layout []
        (Ui.column [ Ui.spacing 12, Ui.padding 16 ]
            [ Ui.el [ Font.size 24, Font.bold ] (Ui.text model.title)
            , Ui.button [] { onPress = Just Save, label = Ui.text "Save" }
            ])
```

The `<main>` landmark element is `Std.Html.mainNode` (not `main`, which would
collide with your program's `main` entry point). Prefer `Std.Ui.Region` for
landmarks anyway.

## Pinned defaults (apply unless the user overrules)

| Concern | Default |
|---|---|
| UI | `Std.Ui` (typed, no CSS). `Std.Html` only for raw markup. |
| Sky.Live navigation | Every internal link is `sky-nav` (`Attr.attribute "sky-nav" ""` on `<a>`). ONE persistent SSE per session; a plain `<a href>` full-reload opens a fresh SSE each page and can freeze the tab. |
| Auth | `Std.Auth` — bcrypt + HS256 JWT cookies. `Auth.login` / `Auth.register` return `Task Error Int` (the user id). Never `fmt`-print a secret. |
| Password forms | `Ui.form [Ui.onSubmit DoSignIn]` with a typed record arg. Never per-keystroke `onInput` on a password field. |
| DB | Records → `Std.Db.Store` + `Std.Codec` (one codec drives JSON **and** dialect-safe DB). SQLite for prototypes, PostgreSQL for multi-instance. Schema via committed file migrations (`sky db migrate --gen`). See **Database** below for the layer choice + the `sky doc` API source of truth. |
| Serialization | `Std.Codec` — **the portable default** for turning a record into JSON and back. ONE codec (`Codec.auto blank`) gives you `Codec.toJson` / `Codec.fromJson` AND, if you persist it, the dialect-safe DB mapping — from a single definition, with no drift between your JSON and DB shapes. Same codec works on every backend (Sky.Live / Http.Server / Cli). Reach for raw `Sky.Core.Json.Encode` / `Sky.Core.Json.Decode` only for a JSON shape a codec can't express — a custom/legacy wire format, or decoding third-party JSON you don't own (there, a hand-written `Decoder` + `Decode.decodeString` is right). |
| Money / decimals | `Std.Money` on `Std.Decimal`. Never raw `Float` for currency. |
| Concurrency | `Cmd.batch` / `Task.parallel`; in-process pub/sub via `Cmd.publish` + `Sub.subscribeTopic`. |
| Errors | `Result Error a` / `Task Error a`. Never `String` as the error type. |
| Logs | `Std.Log` structured logs; `/_sky/console` auto-mounts in dev. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anzellai/sky](https://github.com/anzellai/sky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
