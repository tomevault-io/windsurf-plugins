---
trigger: always_on
description: Guidance for AI coding agents working in the _pywebview_ repository. Human contributors should
---

# AGENTS.md

Guidance for AI coding agents working in the _pywebview_ repository. Human contributors should
start with [docs/contributing/development.md](docs/contributing/development.md) — this file
complements it with the conventions and pitfalls that are easy to miss.

## What this project is

_pywebview_ is a lightweight cross-platform wrapper around the native webview component of the
host OS. Python code creates windows; content is HTML/CSS/JS rendered by the system web engine.
The library ships no GUI toolkit and no browser of its own — it binds to what the platform
already has.

Supported platforms and their renderers:

| Platform | Module | Renderer |
| --- | --- | --- |
| Windows | `webview/platforms/winforms.py` | WinForms host for `edgechromium.py` (WebView2) or `mshtml.py` (deprecated legacy IE) |
| Windows | `webview/platforms/winui3.py` | WinUI 3 host for WebView2 |
| Windows | `webview/platforms/cef.py` | CEF (opt-in, `cefpython3`) |
| macOS | `webview/platforms/cocoa.py` | Cocoa + WKWebView via PyObjC |
| Linux/BSD | `webview/platforms/gtk.py` | GTK 3 + WebKit2 via PyGObject |
| Linux/BSD/any | `webview/platforms/qt.py` | Qt5/Qt6 + QtWebEngine via QtPy |
| Android | `webview/platforms/android/` | Android WebView via pyjnius |

## Repository layout

```
webview/              the library
  __init__.py         public API: start(), create_window(), settings, module state
  window.py           Window class — the user-facing object, delegates to the active backend
  guilib.py           backend detection and import
  util.py             JS injection, the JS↔Python bridge dispatcher, path helpers, _TOKEN
  http.py             built-in Bottle HTTP server (per-window and global)
  event.py            Event / EventContainer
  state.py            window.state — dict synced with window.pywebview.state in JS
  dom/                Python-side DOM API (Element, DOM, classlist, propsdict)
  js/                 JS injected into every page: api.js, customize.js, state.js, finish.js, lib/
  platforms/          one module per backend (see table above)
  lib/                bundled binaries (WebView2 DLLs, Android jar) — do not edit by hand
  __pyinstaller/      PyInstaller hook
interop/              C# (mshtml) and Java (Android) sources for the binaries in webview/lib
tests/                pytest suite — each test opens a real window
examples/             runnable single-file examples, one feature each
docs/                 VuePress site (guide, api, contributing, CHANGELOG)
```

## Installation

_pywebview_ requires Python 3.10 or newer. Install it according to the target platform:

- **Windows:** `pip install pywebview` installs the WinForms dependencies. WebView2 requires the
  Microsoft WebView2 Runtime; use `pip install "pywebview[cef]"` for the optional CEF backend.
  The released WinUI 3 backend can be installed with `pip install "pywebview[winui3]"` and
  requires the Windows App Runtime.
- **macOS:** `pip install pywebview` installs the PyObjC dependencies for Cocoa. The optional Qt
  backend can be installed with `pip install "pywebview[qt]"`.
- **Linux/BSD:** choose a backend explicitly with `pip install "pywebview[gtk]"` or
  `pip install "pywebview[qt]"`. GTK may also require system GTK 3, PyGObject and WebKit2 packages.
- **Android:** use `pip install "pywebview[android]"` and package the application according to
  Kivy's Android packaging workflow.

The `qt5`, `qt6`, `pyside2` and `pyside6` extras select other Qt bindings. The `ssl` extra adds
`cryptography` for HTTPS support in the local server. See `docs/guide/installation.md` for system
package recipes and current platform requirements.

## Architecture rules

**`Window` is backend-agnostic.** `webview/window.py` never imports a platform module. It holds
state and calls `self.gui.<function>(..., self.uid)`, where `self.gui` is the module chosen by
`guilib.initialize()`. Anything platform-specific belongs in `webview/platforms/`.

**Every backend implements the same module-level function contract.** The canonical list is the
set of module-level functions in `webview/platforms/cocoa.py`: `setup_app`, `create_window`,
`get_active_window`, `set_title`, `create_confirmation_dialog`, `create_file_dialog`, `load_url`,
`load_html`, `destroy_window`, `hide`, `show`, `toggle_fullscreen`, `set_on_top`, `resize`,
`maximize`, `minimize`, `restore`, `move`, `get_current_url`, `clear_cookies`, `get_cookies`,
`evaluate_js`, `get_position`, `get_size`, `get_screens`, `add_tls_cert`, plus a module-level
`renderer` string. **Adding a method to `Window` means adding it to every backend** — including
`cef.py`, `mshtml.py` and `android/`, which are easy to forget. If a backend genuinely cannot
support a feature, log a warning and return a sane default rather than raising.

**Windows are addressed by `uid`.** Backends keep their own registry of native windows keyed by
`window.uid`; the Python `Window` object never holds a native handle except `window.native`,
which the backend assigns after creation.

**API calls are gated on lifecycle events.** `Window` methods are decorated with `@_shown_call`,
`@_loaded_call`, `@_before_load_call` or `@_pywebview_ready_call` (see `window.py`). These block

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r0x0r/pywebview](https://github.com/r0x0r/pywebview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
