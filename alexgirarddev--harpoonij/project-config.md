---
trigger: always_on
description: HarpoonIJ is an IntelliJ Platform plugin — a port of ThePrimeagen's NeoVim
---

# Project agent memory

HarpoonIJ is an IntelliJ Platform plugin — a port of ThePrimeagen's NeoVim
[Harpoon](https://github.com/ThePrimeagen/harpoon). It pins files to numbered slots, jumps to them
by hotkey, and shows the list in a popup that behaves like a Vim buffer.

This file is the project's committed home for project-intrinsic agent knowledge: architecture,
sharp edges, the rules that past defects bought at high cost. It owns the IdeaVim story and the
testing bar. `development.md` owns build, toolchain, versions and CI; `README.md` owns the
user-facing feature and keybinding documentation. Do not duplicate across the three.

## How it fits together

One flat package, `src/main/java/ca/alexgirard/harpoonIJ`. `src/main` is Java on purpose —
`todo.md` wants a Kotlin migration eventually; do not mix that into an unrelated change.

- **State** — `HarpoonState`. A static `Map<projectName, List<VirtualFile>>` plus a
  `PropertiesComponent` list under the key `HarpoonJumpList` holding absolute paths. A `null`
  element is an empty slot. Every other class reads and writes the list through it.
- **Settings** — `AppSettingsState` (application service: popup width/height/font size, and
  `enterRemap`), surfaced by `AppSettingsConfigurable` + `HarpoonIjSettingsComponent`.
- **Pin actions** — `SetHarpoonFileAction1..5` over `SetHarpoonFileActionBase` write the current
  file to a fixed slot; `AddToHarpoonAction` writes it to the first empty slot, or appends.
- **Jump actions** — `GotoHarpoon1..5Action` over `GoToHarpoonActionBase` open the file in a fixed
  slot. There are five hotkey slots, but the list itself is not capped at five.
- **Popup** — `ShowHarpoon` (the action) renders the list and shows `HarpoonDialog` (the
  `DialogWrapper`). `NextHarpoonItem` / `PreviousHarpoonItem` / `SelectHarpoonItem` are separate
  actions that forward through static hooks on `ShowHarpoon` to whichever dialog is currently
  showing, and do nothing when none is.
- **IdeaVim** — `IdeaVimIntegration`. The only class in `src/main` that names a
  `com.maddyhome.idea.vim` type. Keep it that way.

Actions are registered in `src/main/resources/META-INF/plugin.xml`; the action IDs there are the
public API users bind in their `ideavimrc`, so renaming one breaks every existing config.

A keystroke becoming a file open, taking `<C-e>` then `<cr>` as the example:

1. IdeaVim (or an IDE keymap binding) invokes the `ShowHarpoon` action.
2. `ShowHarpoon` renders `HarpoonState.GetFiles` one canonical path per line, with the project base
   path abbreviated to `...`, and opens `HarpoonDialog` on that text.
3. `HarpoonDialog` builds an `EditorTextField` over a **real file on disk** (see the next section),
   asks `IdeaVimIntegration` to map `<cr>` to `:action SelectHarpoonItem`, and on the popup's first
   focus event sends Escape so it lands in normal mode with the caret on line 0.
4. `<cr>` fires `SelectHarpoonItem` → `ShowHarpoon.SelectHarpoonItem()` → `HarpoonDialog.Ok()`,
   which records the caret's line in `SelectedIndex` and closes the dialog.
5. `ShowHarpoon` re-reads the popup text through `dialog.getListText()` — the dialog caches it past
   `dispose()`, which is when the backing file is deleted. If the text changed, `SetFiles` rewrites
   the list. Then `NavigateToIndex` opens whatever is in `SelectedIndex`.

The round trip through the popup is lossy: `SetFiles` skips blank and unresolvable lines and only
advances its slot index on lines that resolve, so saving the popup re-packs the list and empty slots
do not survive it. Related to, but distinct from, the persistence gap listed below — that one loses
empty slots when writing to `PropertiesComponent`, this one loses them on the way in.

## The popup must stay backed by a real file

This is the single most expensive thing this project has learned. IdeaVim
[commit `2c057e93`](https://github.com/JetBrains/ideavim/commit/2c057e93) (VIM-3929, 2025-05-23),
first shipped in **IdeaVim 2.25.0 (published 2025-05-27)**, added an unconditional clause to
`EditorHelperRt.isIdeaVimDisabledHere`: IdeaVim now installs no key handling at all in an editor
whose document is not backed by a real file. Before 2.25.0 that check only applied through the
`ideavimsupport` option's `dialog` entry.

The Harpoon popup was an `EditorTextField` over a plain in-memory document, so from 2.25.0 on
IdeaVim attached nothing to it. Typed keys fell through to the platform's ordinary typing handler,
which is indistinguishable from being stuck in insert mode, and `VimShortcutKeyAction` was disabled
so Escape was never claimed by Vim and instead reached `DialogWrapper`'s cancel action. That is one
defect wearing two faces. It broke the plugin's headline feature for over a year, produced three
failed fix attempts in the git log (search for "normal mode") and user reports including
[issue #23](https://github.com/AlexGirardDev/HarpoonIJ/issues/23). Commit `6431caf` fixed it by
backing the popup's document with a real temp file.

What that means for anyone touching `HarpoonDialog`:

- **Never revert the popup to an in-memory document, and do not substitute a `LightVirtualFile`.**
  `EditorHelper.isFileEditor` explicitly rejects a `LightVirtualFile` unless it resolves to a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexGirardDev/HarpoonIJ](https://github.com/AlexGirardDev/HarpoonIJ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
