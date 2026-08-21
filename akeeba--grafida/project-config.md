---
trigger: always_on
description: Grafida is a cross-platform **desktop application** (macOS, Windows, Linux) for creating
---

# Grafida — AI assistant orientation

Grafida is a cross-platform **desktop application** (macOS, Windows, Linux) for creating
and editing **Joomla! articles** through the Joomla Web Services (REST) API. It is built in
**PHP 8.4** with [**Boson**](https://bosonphp.com), uses **SQLite** for all local storage
(via **`joomla/database`**'s `SqliteDriver`, wired through a **`joomla/di`** container),
and **TinyMCE 8** as the HTML editor. Licensed **GNU GPL v3 or later**. Dev happens on macOS.

## Scope (what we deliberately do NOT support)

Grafida is an **offline article editor**: it composes article HTML locally and publishes it
through the REST API. It is **not** the Joomla back-end and does **not** reuse the Joomla
WYSIWYG editor environment. Consequently we do **not** support — and will not try to emulate —
**page builders** (SP Page Builder, JSN, Quix, etc.), **editor-button/editor-xtd plugins**
(e.g. the article/image/page-break/module-insert buttons, sliders, tabs, third-party
shortcode buttons), or **custom/alternative media managers**. Article bodies are plain HTML
authored in TinyMCE; images go through Grafida's own offline media picker (see `src/Media/`),
not a site-side media-manager plugin. Don't add features that depend on server-side editor
plugins or builder shortcodes.

## How Boson works here (important)

Boson runs a native webview and bundles a PHP runtime. There is **no `webview->bind` RPC**.
Instead `index.php` registers a handler for the `boson://` scheme; every request is turned
into a PSR-style `Boson\Component\Http\Request` and answered with a `Response`. The
front-end (a plain HTML/CSS/JS SPA under `assets/private`) talks to PHP by calling
`fetch('boson://app/api/...')`.

Request flow: `index.php` → `Grafida\FrontController` → `Grafida\Application\Kernel` →
either `Grafida\Http\ApiController` (paths under `/api/`) or a static asset / the SPA shell.
**The kernel is a pure `Request → Response` function**, so the whole back-end is testable
without opening a window (see `tests/Feature/ApiRoutingTest.php`).

**The composition root is a DI container, not the Kernel.** `index.php` builds a
`Grafida\Application\Container` (a thin `Joomla\DI\Container` subclass — see `src/Application/`)
via `ContainerFactory::create()` and pulls `FrontController` out of it; `Kernel` is now just
`(StaticProviderInterface $static, ApiController $api)`. Nothing is `new`ed ad-hoc and there is
no global/singleton database object — add a service by registering it in a **service provider**
(`src/Application/Provider/`), not by editing a constructor chain.

**File pickers must go through the native dialog, not `<input type="file">`.** Boson's
webview does not wire up the HTML file-input open-panel callback (WKWebView on macOS,
WebKitGTK on Linux), so an in-page `<input type="file">` `.click()` silently does nothing.
`index.php` therefore passes `$app->dialog` (Boson's `DialogApiInterface`) into the container
as the **`dialog` parameter**, from where it reaches `SettingsController`, and the SPA opens
files via `POST /api/dialog/open-file` (`api.openFile(filter)`, filter
`image`/`markdown`/`any`): `SettingsController::openFile()` calls `selectFile()`, reads the
chosen file and returns `{name, mime, dataBase64}` (or `{cancelled:true}`). `uploadLocalImage()`
(intro/full-text images, the in-editor/media-browser "Choose file…" button) and
`importMarkdown()` consume it. The dialog dependency is **nullable** so the kernel stays
window-free in tests (a null dialog makes the endpoint return 503).

## Layout

- `src/Application/` — the **composition root**. `Container` is a thin `Joomla\DI\Container`
  subclass whose only job is to give `get()` a generic return type (the parent's is `mixed`,
  which PHPStan level max cannot use). `ContainerFactory::create(array $parameters = [])`
  registers the parameters, then the five `Provider/` service providers (`StorageProvider`,
  `HttpProvider`, `SiteProvider`, `AiProvider`, `AppProvider`, `ControllerProvider`). The
  parameters are the app's only configuration seams — override them and you get a different
  app without touching a constructor:
  `db.path` (default `Paths::databaseFile()`; `':memory:'` in tests), `migrations.dir`,
  `base.path`, `docs.dir` (default `Resources::docsDir()` — note it does **not** go through
  `Resources::base()`, since the docs are read straight out of the phar rather than extracted),
  `static.provider`, `dialog` (nullable), and `secret.store` — which is
  **tri-state**: `null` → `SecretStoreFactory::secureStore()` (production), `false` → no store
  (forces the insecure-plaintext fallback path), a `SecretStore` instance → used as-is.
  The `DatabaseInterface` factory **connects *and* migrates**, so every consumer receives a
  migrated database. `Kernel` is `(StaticProviderInterface, ApiController)`.
  It also holds the two classes that make the **native event loop** affordable: `BosonApplication`
  (the `Boson\Application` `index.php` actually instantiates) and `EventLoopThrottle`.
  ⚠️ **Boson's event loop is a busy-wait and costs about half a CPU core with the app idle.**
  `Application::run()` calls `$poller->next()` forever; the stock poller separates two iterations by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akeeba/grafida](https://github.com/akeeba/grafida) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
