---
trigger: always_on
description: Small GPU-accelerated text/code editor in Odin. SDL3 + SDL3_ttf for
---

# Bragi

Small GPU-accelerated text/code editor in Odin. SDL3 + SDL3_ttf for
window/text, libvterm + forkpty for the embedded terminal pane.
Modal (vim) editing, hand-rolled syntax highlighting, side-by-side
panes, native dialogs, theme-able chrome.

User-facing docs (features, key bindings, build instructions,
packaging) live in `README.md`. This file is for LLMs working on the
code: architectural invariants, file map, and decisions that aren't
self-evident from reading the source.

## Build

```
odin build .                  # produces ./Bragi
./Bragi [path/to/file]
```

Requires Odin **dev-2026-04** or newer (`core:os` overhaul). Runtime
deps: `sdl3`, `sdl3_ttf`, `libvterm` (Homebrew on macOS, distro
packages on Linux). On Windows, ship `SDL3.dll`, `SDL3_ttf.dll`, and
`vterm.dll` next to the produced `Bragi.exe`. The Windows libvterm
build is vendored under `vendor/libvterm/` (built from neovim's
libvterm fork — vcpkg has no port for it). Re-run
`vendor/libvterm/build.ps1` to refresh the binaries; that script
clones the upstream repo into `vendor/libvterm/_src/`, drops in a
small CMakeLists.txt, builds with MSVC, and copies the resulting
`vterm.dll` / `vterm.lib` / headers back into `vendor/libvterm/`.

The fff file-search library (powers the finder) is vendored under
`vendor/fff/` as prebuilt per-arch `c-lib-*` binaries — it has no
Homebrew/distro/vcpkg port. Unlike the deps above it's bundled into the
release on every platform (macOS Frameworks/, Linux `/usr/lib/bragi/`,
next to `Bragi.exe`). macOS dylibs need an `install_name` rewrite to
`@loader_path/...`; Windows needs an import lib generated from the DLL.
Both are documented in `vendor/fff/README.md`. Only the macOS path is
verified end-to-end so far.

Two TTFs are embedded via `#load`:
- `FiraCode-Regular.ttf` → editor pane (`g_font`)
- `FiraCodeNerdFont-Regular.ttf` → terminal pane (`g_terminal_font`)

Both have identical advance width so cell math is unchanged.

## File map

- **`main.odin`** — SDL init, main loop, layout, input dispatch, theme,
  text cache, native dialogs, pane lifecycle, draw orchestration.
- **`editor.odin`** — `Editor` struct, cursor / selection, edit
  primitives, auto-close brackets, smart Enter, soft tabs.
- **`piece_buffer.odin`** — Piece-list backing store for `Editor.buffer`.
  Immutable `original` (file load) + append-only `added` + ordered
  list of pieces; far cursor jumps are O(piece-list-edit) instead of
  O(distance). Sequential-access cache makes consecutive `byte_at`
  calls O(1) amortized; coalesces on typing runs. Same `version: u64`
  contract as before for cache keys. `original` is either heap-
  allocated or an mmap'd region; `_destroy` dispatches on
  `original_mmap`.
- **`mmap_posix.odin`** / **`mmap_other.odin`** — POSIX mmap loader
  used by `editor_load_file`. `MAP_PRIVATE | PROT_READ | PROT_WRITE`
  so CRLF compaction can run in place via copy-on-write without
  modifying the file. Pure-LF files (the common case) touch nothing
  and stay lazy-paged. Windows / other platforms get a stub that
  returns `ok=false` so the load path falls back to read-into-buffer.
- **`undo.odin`** — Edit-log undo/redo with adjacent-op merging.
- **`file.odin`** — Load (direct-into-gap-buffer + EOL detect) / save
  (atomic, EOL expand) / `path_basename` / `digit_count`.
  **Saves MUST be atomic on POSIX** (`write_file_atomic`: write a sibling
  `.bragi-tmp` + `rename` over the target), NOT an in-place
  `write_entire_file`. The buffer keeps reading its `MAP_PRIVATE` mmap of
  the loaded file for the file's lifetime; truncating+rewriting that same
  inode in place makes any piece pointing past the new EOF read back as
  zeros on Linux (macOS' VM happens to keep the old resident pages, so it
  never reproduced there — symptom was "Ctrl+S adds an empty line each
  save"). Rename leaves the original inode unlinked-but-alive behind the
  mapping, so pages stay valid; it also follows symlinks (preserves the
  link via `stat().fullpath`) and keeps the target's mode. Windows uses the
  read-into-buffer load path (no mmap), so it keeps the in-place write.
  Regression test: `save_test.odin`.
- **`vim.odin`** — `Mode` enum, vim parser FSM, motions / operators /
  ex commands. Modes: `Insert`, `Normal`, `Visual`, `Visual_Line`,
  `Command`, `Search`.
- **`syntax.odin`** — Per-language tokenizers (Odin / C / C++ / Go /
  Jai / Swift / Rust / V / GDScript / Bash / INI / Markdown / Generic / None).
  Most go through `tokenize_with_spec`; INI and **Markdown** have their own
  dedicated tokenizers (markup / config don't fit the C-family
  `Language_Spec`). `tokenize_markdown` is line-oriented + inline spans.
  `Tokenizer_State` is a **struct** (`mode` + `fence_lang` + `fence_inner`),
  not a bare enum, so a fenced block remembers its body language and that
  embedded tokenizer's own block-comment mode — letting ```` ```odin ````/
  ```` ```c ```` bodies highlight in their language across lines (via a
  recursive `syntax_tokenize` call; unknown/unsupported tags fall back to flat
  `Md_Code`). `mode` also carries `Fenced_Code` / `Table` across lines. Covers
  headings, emphasis, inline + fenced code, links/autolinks, lists, task
  checkboxes (`- [ ]`/`- [x]` → `Md_Task`), blockquotes, rules, tables, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Galaxoid-Labs/Bragi](https://github.com/Galaxoid-Labs/Bragi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
