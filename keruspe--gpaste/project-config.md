---
trigger: always_on
description: This file covers both the library under `src/libgpaste/gpaste-daemon/` and the executable under `src/daemon/`. The D-Bus interface rules it implements are in [`src/libgpaste/AGENTS.md`](../AGENTS.md).
---

# `src/daemon/` — `gpaste-daemon` + **libgpaste-daemon**

This file covers both the library under `src/libgpaste/gpaste-daemon/` and the executable under `src/daemon/`. The D-Bus interface rules it implements are in [`src/libgpaste/AGENTS.md`](../AGENTS.md).

Most of the rationale behind what is described here is written as comments beside the code it explains; this file says what exists, where it lives, and the invariants that span several files. Read the comments of a function before changing it.

## Layout

The background service owns the clipboard history and exposes it over D-Bus (`org.gnome.GPaste`).

- Almost all of its objects live in the installed, introspectable **libgpaste-daemon** library: sources under `src/libgpaste/gpaste-daemon/`, umbrella header `src/libgpaste/gpaste-daemon.h`, `GPasteDaemon-1` GIR/typelib. Its types keep the `GPaste`/`g_paste_` prefix, so the GIR passes an explicit `identifier_prefix: 'GPaste'` / `symbol_prefix: 'g_paste'` to place them in the `GPasteDaemon` namespace.
- **Headers are split in `meson.build`**, and the distinction is real. `gpaste_daemon_public_headers` are installed, introspected and included by the `gpaste-daemon.h` umbrella: seven because the extension drives them (`GPasteBus`, `GPasteDaemon`, `GPasteSearchProvider`, `GPastePrompt` — which `prompt.js` implements — `GPastePassphrase`, `GPasteStorageMigration`, and `GPasteStorageBackend` for its static passphrase helpers), and eight more only because those pull them in (`g_paste_daemon_new()` needs the clipboard provider, hence the history, hence the items). `gpaste_daemon_internal_headers` are compiled in but **never installed nor introspected**; in-tree consumers (`src/daemon/`, `src/ui/`, `tests/`) reach them through `gpaste_daemon_headers_dep`'s `include_directories`. The optional features append to the internal lists, so the GIR does not change shape with the feature set. **A new header is internal unless the extension needs it**: the umbrella may only include installed headers, and a public header is a commitment in the GIR.
- The utility helpers nothing outside the library calls live in `gpaste-daemon-util.h` (the history-path helpers, the file backend's XML escaping). `g_paste_util_history_name_is_valid()` is not one of them: a client asks it too before sending a name, so it is installed in `gpaste-3/gpaste-util.h` beside `G_PASTE_DEFAULT_HISTORY`.
- `src/daemon/gpaste-daemon.c` is the thin executable entry point that links it. What stays in `src/daemon/` is what only that executable uses: the **GDK clipboard backend** (`gpaste-clipboard-gdk.{c,h}` and its `gpaste-text-content-provider.{c,h}`) — which is why `gtk4-x11` is a dependency of the executable and not of the library — and the Adwaita prompt (below). They are compiled straight into the binary, include each other with quoted local includes, and carry no `G_PASTE_VISIBLE`: nothing exports them.

## Items, keybindings and the history model

- **Clipboard watching** (primary + clipboard selections) goes through the backend-agnostic `GPasteClipboardProvider` interface (see Clipboard backends).
- **Items**: the abstract `GPasteItem` base and the five kinds deriving from it directly — `GPasteTextItem`, `GPastePasswordItem`, `GPasteColorItem`, `GPasteImageItem`, `GPasteUrisItem` — plus the `GPasteSpecialMime`/`GPasteBinaryData` helpers and `GPasteSensitiveMime`. The UI and client use libgpaste's lightweight `GPasteClientItem` instead. **The hierarchy is flat on purpose**: a kind deriving from `GPasteTextItem` without inheriting anything from it would make `G_PASTE_IS_TEXT_ITEM()` true for something that is not text, and would force `g_paste_item_equals()` to dispatch both ways. The base's `equals` compares kind and value, which is symmetric; only the image (checksum) and the password (a named one never matches, a nameless one matches by value) override it.
- **Keyboard shortcuts** are registered through the XDG GlobalShortcuts portal alone (`GPasteGlobalShortcutClient`, used directly by `GPasteKeybinder`); when the portal is unavailable, they are disabled. The keybinder, whose comments carry the reasons (`gpaste-keybinder.c`):
  - honours the `keybindings-enabled` master switch by handing the client an empty set, caching in `self->enabled` the value it last applied (seeded in `g_paste_keybinder_new()`), and rebinding only when the setting no longer matches it;
  - debounces rebinds on a 250 ms timeout re-armed by every write, the source holding the keybinder weakly;
  - deactivates every keybinding before activating them (edited accelerators are only re-parsed that way), but never ungrabs: `grab_all()` replaces the client's whole set in one call, which is what lets it recognise an unchanged set and stay put. That is also why a switch flipped and flipped back, or an accelerator written back to its own value, needs no special case — and why there is no public `deactivate_all()` counterpart to `g_paste_keybinder_activate_all()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Keruspe/GPaste](https://github.com/Keruspe/GPaste) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
