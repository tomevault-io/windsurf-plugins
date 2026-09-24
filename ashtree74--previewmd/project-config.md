---
trigger: always_on
description: PreviewMD is a native, professional macOS Markdown reader created, owned, and
---

# PreviewMD Project Instructions

## Product

PreviewMD is a native, professional macOS Markdown reader created, owned, and
maintained independently by Adam Jesionkiewicz (`adam@jesion.pl`). It is
licensed under Apache-2.0. Astrography Sp. z o.o. has no ownership or governance
rights in the project; its Apple Developer account is currently used only to
sign and distribute macOS builds under the same public license as everyone
else.

- Keep the interface native, minimal, and consistent with current macOS design.
- The application targets macOS 14 or newer.
- Release builds must be Universal 2 (`arm64` and `x86_64`).
- The bundle identifier is `pl.jesion.previewmd`.
- The exported Markdown UTI is `pl.jesion.previewmd.markdown`.
- The About window must show the author, email, and copyright year 2026.
- The source code is Apache-2.0 and external contributions use DCO 1.1. Keep
  `LICENSE`, `NOTICE`, `AUTHORS.md`, and the original-author attribution intact.
- Every assembled `.app` must include the project `LICENSE`, `NOTICE`, and
  `THIRD_PARTY_NOTICES.md` under `Contents/Resources/Legal`; the About window
  must continue to identify Apache-2.0 and the original author.
- The PreviewMD name and icon identify the official project. Forks and public
  ports follow `TRADEMARKS.md`; a signing, hosting, or distribution provider
  does not gain ownership or project authority.
- The About window must also carry the bundled renderers' full license texts
  and copyright lines. The MIT and BSD licenses require those notices to ship
  with every copy, so they are compiled into the binary
  (`Acknowledgements.swift`), never left to a resource file or a repo-level
  Markdown file that packaging can silently drop. Upgrading a vendored library
  means updating its version and license text there in the same commit.

## Product invariants

Preserve these behaviors unless the user explicitly requests a change:

- A clean launch opens an empty state, not a default README or demo document.
- The empty state uses the PreviewMD application icon.
- The embedded showcase/demo must work on every supported Mac without relying
  on files from the development machine.
- Closing the final document tab returns to the empty state.
- Files opened from Finder or dropped on the Dock icon open as tabs in the
  existing main window rather than creating unnecessary windows.
- Drag and drop works across the main window.
- The left sidebar is collapsed by default.
- When opened, the left sidebar should start near 200 pt and remain compact
  like the sidebar in Preview rather than consuming document-reading space.
- Opening a folder shows a collapsible tree of its supported Markdown and text
  documents in the existing left sidebar. Empty/irrelevant directories, hidden
  files, packages, and symlinked directories stay out of the tree.
- Folder loading must not block the main thread, and opening a folder must not
  automatically open a README or any other document. Files selected from the
  tree open as tabs in the existing main window.
- The document inspector should also open near 200 pt, expanding only enough
  to keep outline and insight content usable.
- The document inspector is collapsed by default.
- Use the regular unified macOS toolbar, with a height comparable to Preview;
  do not switch the main window back to the compact toolbar style.
- Let the system own the sidebar and inspector surfaces. Do not stack custom
  translucent materials on top of `NavigationSplitView` or `.inspector`.
- Do not force a single opaque background across the window toolbar; the
  sidebar surface must continue through the title bar like system macOS apps.
- On macOS 26 and newer, keep the neutral detail background eligible for
  `backgroundExtensionEffect()` so the system sidebar can sample content.
- Paper canvas is disabled by default.
- Documents open immediately with the current saved appearance settings; do
  not briefly render with paper canvas or another default layout first.
- Reading width can be adjusted continuously with a clean width control.
- Narrow, comfortable, wide, and table/data presets remain available in the
  menu.
- Tables, Mermaid diagrams, KaTeX, code highlighting, local images, and
  relative links must continue to render offline.
- The application bundle includes a sandboxed Quick Look preview extension at
  `Contents/PlugIns/PreviewMDQuickLook.appex`. It supports both
  `pl.jesion.previewmd.markdown` and the common
  `net.daringfireball.markdown` UTI, renders with bundled resources, and must
  never require a separate plugin installation.
- Copying PreviewMD to Applications and opening it once must be sufficient to
  register Quick Look. Do not require users to run `pluginkit`, `qlmanage`, or
  another command.
- PreviewMD declares both Markdown UTIs as editable document types with the
  `Default` handler rank. Settings provides an explicit action to make
  PreviewMD the default Markdown app, which also controls Quick Look's Open
  button. Never silently replace a user's existing default application.
- Quick Look must replace inaccessible document-relative images with a clear
  open-in-PreviewMD message, never a broken-image glyph. This Finder sandbox

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ashtree74/PreviewMD](https://github.com/ashtree74/PreviewMD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
