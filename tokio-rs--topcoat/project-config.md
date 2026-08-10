---
trigger: always_on
description: Topcoat is a Cargo workspace. The framework crates live in `crates/`, small single-feature examples in `examples/`, complete demo applications in `demos/`, and the prose guides in a `docs/` directory inside the crate they document.
---

# Agent instructions

## Project structure

Topcoat is a Cargo workspace. The framework crates live in `crates/`, small single-feature examples in `examples/`, complete demo applications in `demos/`, and the prose guides in a `docs/` directory inside the crate they document.

`crates/topcoat` is the user-facing **facade** crate. It re-exports everything through feature-gated modules. Application code depends on this crate only; everything below is an implementation detail reached through it.

- `topcoat-core`: foundations shared by the other crates: the `Error`/`Result` types and the request context (`Cx`, `app_context`, `request_context`). Its macro crate provides `#[memoize]`, and its grammar crate holds the pretty-printer backing `topcoat fmt`'s macro-body formatting.
- `topcoat-view`: the `view!`, `attributes!`, and `class!` macros, the `#[component]` macro, and the runtime `View`/`Attributes`/`Class` types.
- `topcoat-router`: `Router`, the `#[page]`/`#[layout]`/`#[route]` macros, `module_router!`, `path_param!`, and `#[query_params]`.
- `topcoat-runtime`: the client-side interactive runtime (signals, event handlers, bind attributes, the `expr!` macro) and the injected browser script.
- `topcoat-font`: the `font!` and `font_face!` macros and the Fontsource integration for bundling and serving web fonts.
- `topcoat-icon`: the `icon` component and the Iconify integration for vendoring icon sets into a project.
- `topcoat-asset`: the `asset!` macro and `AssetBundle` for declaring and serving content-hashed static files.
- `topcoat-cookie`: the cookie jar, `cookie!` macro, signed/private jars, and `CookieStore<T>`.
- `topcoat-session`: bring-your-own-storage session authentication: the token/hash model, the session lifecycle, and origin checking.
- `topcoat-mail`: the `Mail` type and `mail!` macro for declaring mail, and its delivery through pluggable transports (SMTP, file, in-memory).
- `topcoat-htmx`, `topcoat-alpine-ajax`, and `topcoat-datastar`: request and response helpers for those client libraries.
- `topcoat-tailwind`: the build-script wrapper around the standalone Tailwind CLI.
- `topcoat-ui` (+ `registry/`): the component registry behind `topcoat ui`, which copies component source into a project.
- `topcoat-cli`: the `topcoat` binary. Each subcommand has its own module under `src/`.

A crate that backs proc-macros comes as a trio. The base crate holds the runtime types the generated code calls into. Its `grammar/` crate parses the macro body and generates the code, and is only used at compile time. Its `macro/` crate is a thin proc-macro entry point over `grammar/`. Where a macro body is formattable, the `grammar/` crate's `pretty` feature adds the pretty-printer `topcoat fmt` uses.

## Documentation

Each crate's `docs/` directory holds the user-facing guides for that crate, embedded into the API docs with `#![doc = include_str!(...)]`. Consult the relevant one before working on a feature in that area. The index below covers the main guides; when working on a crate, check its own `docs/` directory for anything not listed here.

### Getting started

- [`crates/topcoat/docs/getting_started.md`](crates/topcoat/docs/getting_started.md): Creating a new project, installing the `topcoat` CLI, and running the dev server.

### Routing

- [`crates/topcoat/docs/router.md`](crates/topcoat/docs/router.md): The `Router` primitive: registering `#[page]`, `#[layout]`, and `#[route]` items manually or via `.discover()`, and how layouts nest by path prefix.
- [`crates/topcoat-router/docs/module_router.md`](crates/topcoat-router/docs/module_router.md): `module_router!`, which derives routes from the module tree (kebab-cased segments, `segment!` overrides, `_`-prefixed groups).
- [`crates/topcoat-router/docs/error.md`](crates/topcoat-router/docs/error.md): Router errors: the status-code constructors, the `RouterErrorExt` conversions from `Option`/`Result`, and catching an error in an outer handler.
- [`crates/topcoat-router/docs/tower.md`](crates/topcoat-router/docs/tower.md): The tower bridge: `TowerRoute` for mounting a tower service as a route and `TowerLayer` for running tower middleware as a layer.
- [`crates/topcoat-router/docs/content.md`](crates/topcoat-router/docs/content.md): Request and response bodies: `FromRequest` extractors, `IntoResponse` return values, and an overview of the content types below.
- [`crates/topcoat-router/docs/content/websocket.md`](crates/topcoat-router/docs/content/websocket.md): WebSockets (behind the `websocket` feature): the `WebSocketUpgrade` extractor, exchanging `Message`s over a `WebSocket`, subprotocol negotiation, and connection limits.
- [`crates/topcoat-router/docs/content/sse.md`](crates/topcoat-router/docs/content/sse.md): Server-sent events (behind the `sse` feature): the `Sse` streaming response, building `Event`s, keep-alive events for idle streams, and resuming from `Last-Event-ID`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tokio-rs/topcoat](https://github.com/tokio-rs/topcoat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
