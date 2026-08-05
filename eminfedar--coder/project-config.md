---
trigger: always_on
description: VSCode-like code editor in terminal. Rust + ratatui + tokio. Elm Architecture (Model / Msg / Cmd / update / view). **Code comments and UI text are in English.**
---

# AGENTS.md

VSCode-like code editor in terminal. Rust + ratatui + tokio. Elm Architecture (Model / Msg / Cmd / update / view). **Code comments and UI text are in English.**

## Running

```bash
cargo run [dir]          # opens cwd if no dir is given
cargo build              # dev
cargo test               # unit tests (core/buffer, services/git)
cargo clippy             # lint (kept warning-free)
```

Environment: `CODER_ASCII=1` uses ASCII instead of Nerd Font icons. `CODER_EXTENSIONS` overrides extensions dir (default `~/.config/coder/extensions`).

## Architecture (Elm Architecture)

Single unidirectional loop, `src/main.rs::run()`:

```
event / async result  ->  Msg  ->  update(model, msg) -> Vec<Cmd>  ->  cmd::execute -> Msg (via tx)
                                        |
                                     Model changes  ->  ui::view(frame, model)
```

- **Model** (`app/model.rs`): all state. Pure data, no IO.
- **Msg** (`app/msg.rs`): input events + async results. Only input to `update`.
- **update** (`app/update/`): takes `Msg`, mutates `Model`, returns side-effect `Cmd`s. Synchronous, no IO.
- **Cmd** (`app/cmd.rs`): description of side effect. `execute()` runs on tokio, sends result back as `Msg` via `tx`.
- **view** (`ui/mod.rs`): draws `Model` with ratatui. Read-only.

Rule: IO and async live only inside `cmd::execute`. `update` never blocks (exception: `ReplaceDone` reads open buffers from disk with short synchronous `std::fs::read_to_string`).

### Layout sharing
`ui::compute_areas(model, area) -> Areas` computes layout once; both `view` and mouse hit-testing (`update::handle_mouse`) use same `Areas`. Mouse coordinate mapping always goes through it.

## Directory map

| Path | Role |
|------|------|
| `main.rs` | tokio runtime, terminal setup/restore, event loop, `map_event`, `spawn_watcher` (filesystem watcher) |
| `app/model.rs` | `Model`, `Sidebar`, `Tab` (+ `head_text`, `diff_mode`, `title()`), `SearchState`, `GitStatus`, `SettingsState`, `LayoutState`, `Panel`, `Focus`, `DragTarget`, `Dialog`/`DialogKind`/`DialogAction`. `refresh_git_marks()`/`git_gutter()`/`active_git_marks` (change gutter), `active_is_diff()`/`diff_tab_index_for()`/`pending_diff` (diff tab), `DiffRow`/`active_deleted`/`diff_rows()`/`diff_start()`/`screen_row_to_line()` (inline deleted-line view) |
| `app/msg.rs` | `Msg` enum |
| `app/update/` | Elm `update` layer, split into submodules by concern. `mod.rs` = `update()` Msg dispatcher + shared `use`s (submodules pull them via `use super::*`; `mod.rs`'s private `use` re-exports visible to child modules). Cross-module helpers `pub(super)`, local ones `fn`. Submodules: `action.rs` (`apply_action()`), `editor.rs` (edit/motion/clipboard/viewport + format-on-save), `tabs.rs` (panel select, tab lifecycle, `open_path`/`open_diff`, `reload_if_clean`/`apply_reload`), `sidebar_nav.rs` (`nav`/`activate_selection`/`persist_config`/`new_entry_dialog`), `git.rs` (`git_commit`), `find.rs` (editor find/replace + tests), `search.rs` (search-panel run/replace `Cmd`s), `dialog.rs` (modal key/mouse), `mouse.rs` (mouse routing + hit-test), `terminal.rs` (terminal size), `lsp.rs` (LSP document lifecycle: `open_tab`/`did_save`/`did_close`, debounced `notify_change`/`change_due`, `request_completion`/`completion_key`, `request_format`/`format_edits_arrived`, `store_diagnostics`/`run_linter`). Only `update()` `pub`. Keyboard input to focused text field routed by `focused_input()` (in `mod.rs`) straight to that field's `TextInputState::handle_key`; only keys widget *ignores* (Enter/Tab/Esc, Ctrl-shortcuts, match/result nav) fall through to `keymap::resolve`. |
| `app/cmd.rs` | `Cmd` enum + `execute()` (tokio spawn) |
| `core/buffer.rs` | rope-based text buffer: cursor, selection (anchor), undo/redo, dirty, version. `insert_newline` auto-indents: carries current line's leading whitespace (clipped at cursor) onto new line, inserting break + indent as one `insert_str` so undo takes both. |
| `core/keymap.rs` | `(KeyEvent, Focus) -> Action`. VSCode-style shortcuts |
| `core/highlight.rs` | syntect syntax highlight (`HlLine`). Theme picker: `theme_names()`, `theme_for(name)` (derives UI palette from syntect `settings`), `Highlighter::set_theme`, `DEFAULT_THEME`. `THEME_SET` = syntect defaults + embedded `PALETTES` (`build_theme`) + `theme_dirs()` (.tmTheme folders) |
| `core/filetree.rs` | lazily-loaded file tree |
| `core/theme.rs` | color palette (`Theme` struct + `Default`) |
| `core/text_input.rs` | shared text-input **state**: owned `String` + char-index caret, all edit/motion ops (insert/backspace/delete/left/right/word/home/end/up/down, line-aware so single- and multi-line both work). `handle_key(key, multiline) -> InputOutcome` (`Ignored`/`Moved`/`Changed`) does whole keymap for focused field. Used by find query/replace, workspace search query/replace, git commit. |
| `ui/mod.rs` | `compute_areas`, `view`, `Areas` |
| `ui/find.rs` | editor-embedded find/replace widget (floats top-right). `layout`/`render`/`hit` (`FindHit`). 1-cell inner padding every side (outer box = content+2). Input rows drawn by shared `ui::text_input::TextInput` widget. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eminfedar/coder](https://github.com/eminfedar/coder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
