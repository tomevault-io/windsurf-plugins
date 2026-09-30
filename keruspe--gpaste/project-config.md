---
trigger: always_on
description: Three libraries, each exporting what is marked `G_PASTE_VISIBLE` (hidden default visibility) and each with GIR and Vala bindings:
---

# `src/libgpaste/` — shared libraries

Three libraries, each exporting what is marked `G_PASTE_VISIBLE` (hidden default visibility) and each with GIR and Vala bindings:

- `gpaste-3/` → **libgpaste**: the daemon-agnostic types — `GPasteClient` (the D-Bus client), `GPasteClientItem` (an item as it travels over D-Bus), `GPasteSettings` (the GSettings wrapper), enums and utilities.
- `gpaste-gtk4/` → **libgpaste-gtk4**: GTK4 + Adwaita helpers, the preferences widgets.
- `gpaste-daemon/` → **libgpaste-daemon**: the daemon's objects, documented in [`gpaste-daemon/AGENTS.md`](gpaste-daemon/AGENTS.md).

## Headers, layout and pkg-config

- **Every library splits its headers into an installed half and an internal one**, and a new header joins the internal half unless something outside the library names it. libgpaste-gtk4 installs exactly two types — the preferences dialog, and the preferences widget `prefs.js` embeds — while its groups, shortcut row and pages stay internal; its GIR is generated from `libgpaste_gtk4_public_sources` alone. The four pages are plain functions returning an `AdwPreferencesPage`, not types: they carry no state, and both callers build them from one list (`g_paste_gtk_preferences_pages_new()`). libgpaste's own lists are in `src/libgpaste/meson.build`, and libgpaste-daemon's split is described in its own `AGENTS.md`.
- **The core library's directory carries `apiversion`**, so it is renamed on every major bump (`gpaste-2/` → `gpaste-3/` for 51.0), and its includes are spelled `<gpaste-3/gpaste-macros.h>`. The same include text resolves in-tree (through `include_directories('.')` = `src/libgpaste`) and against the install prefix, which is what makes a broken installed header a build failure here rather than downstream. `gpaste-gtk4/` and `gpaste-daemon/` are named for their library and never move.
- **The installed layout** is one shared directory:

  ```
  include/gpaste/{gpaste.h, gpaste-gtk4.h, gpaste-daemon.h}   <- the three umbrellas
  include/gpaste/{gpaste-3/, gpaste-gtk4/, gpaste-daemon/}    <- per-library headers
  ```

  so all three `.pc` files declare `Cflags: -I${includedir}/gpaste` (meson `subdirs: 'gpaste'`), the line to check when the layout changes. `libgpaste` is versioned all the way through from `apiversion` (`libgpaste-3.so`, `gpaste-3.pc`, `GPaste-3`, `gpaste-3.vapi`); `libgpaste-gtk4`'s `4` is GTK's and `GPasteDaemon-1`'s is its own, so neither tracks the GPaste major.
- **Each library ships a `.pc`** (`gpaste-3`, `gpaste-gtk4`, `gpaste-daemon`). `requires:` is what a *consumer* must also satisfy — what the installed headers name, not what the library links — so `gpaste-daemon` requires only `gpaste-3`: no public daemon header names a GTK, GDK or GCR type, and those stay in the `Requires.private:` meson derives from `dependencies:`, with the optional libsodium, sqlite3, libsecret-1 and libmutter (`pwquality` is on `gpaste-3`, where the rating lives). Use `requires:`, never `libraries:`, which means `Libs:`.

## Settings

**Observe a setting with `notify::<key>`.** Every setting is a GObject property named exactly like its GSettings key, so there is no separate `changed` signal and `g_object_bind_property()` works directly; the callback takes a `GParamSpec *`, not the key. Prefer the detailed form: undetailed `notify` fires for all twenty-nine keys. `GPasteSettings`' one signal of its own, **`rebind::<key>`**, is an action to take (re-register a keybinding), carried only by the keybinding settings.

## D-Bus

Every interface GPaste speaks is described by an XML file that `gdbus-codegen` turns into both halves of the wire. **`data/dbus/org.gnome.GPaste3.xml` is the contract**: its comments document every method, signal and property, and are the first thing to read and to update when the interface changes. What follows is how the tree is built around it.

- **The XML is the source of truth**, installed to `$datadir/dbus-1/interfaces` (`dbus-1.pc`'s `interfaces_dir`, overridable with `-Ddbus-interfaces-dir`). `src/libgpaste/gpaste-daemon/org.gnome.Shell.SearchProvider2.xml` is gnome-shell's interface, kept only to feed codegen and deliberately **not** installed.
- **The generated code is internal**: produced by `gnome.gdbus_codegen()` from `src/libgpaste/gpaste-3/meson.build` and `src/libgpaste/gpaste-daemon/meson.build` (which exist so the output lands where the `<gpaste-3/…>` include style works), never installed, never introspected. `GPasteClient` is the whole public story. `grep -c Daemon3 build/src/libgpaste/GPaste-3.gir` must stay `0`.
- **`org.gnome.GPaste3` is generated into libgpaste**, the lowest library, because a GType registers once per process and gnome-shell loads libgpaste and libgpaste-daemon together. libgpaste-daemon uses the skeleton, so the declarations carry `G_PASTE_VISIBLE` through `--symbol-decorator`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Keruspe/GPaste](https://github.com/Keruspe/GPaste) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
