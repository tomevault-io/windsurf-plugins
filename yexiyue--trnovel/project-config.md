---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 开发工作流

**IMPORTANT**：执行任何开发任务（编写代码、修改配置、添加依赖）前，必须先调用 `/dev-workflow` skill。它会加载项目知识库（`dev-notes/knowledge/`）中的最佳实践和踩坑记录，并在开发完成后引导更新知识库。

知识库主题：

- `dev-notes/knowledge/tui-ratatui-kit.md` — ratatui-kit 的 hooks 求值时机（狂闪 bug）、键位约定、路由
- `dev-notes/knowledge/booksource.md` — 书源规则 DSL、render-fetcher、番茄签名/翻页、反爬、novel-tts
- `dev-notes/knowledge/toolchain.md` — Cargo workspace（mod.rs 风格）、feature 门控、依赖钉版、构建/发布坑

## Overview

TRNovel is a terminal novel reader (Rust, edition 2024) built on `ratatui` + a custom React-like TUI framework crate, `ratatui-kit`. It reads local `.txt` novels and network novels (via Legado-style "book sources"), tracks reading history, supports color themes, and offers TTS playback using the Kokoro model. UI text and code comments are in Chinese.

## Workspace layout

Cargo workspace with one root binary crate and two library members:

- **`trnovel`** (root, `src/`) — the app. Builds two identical binaries: `trnovel` (default) and `trn`.
- **`crates/parse-book-source`** — fetches/parses network novels from Legado-compatible book-source JSON.
- **`crates/novel-tts`** — Kokoro TTS engine wrapper + streaming audio pipeline.

## Commands

```bash
# Build / run
cargo build                       # debug; produces both trnovel and trn
cargo run                         # runs trnovel (default-run); opens the Home page
cargo run -- -q                   # quick: resume last reading position
cargo run -- -l <PATH>            # local novels from a directory
cargo run -- -n                   # network mode  | -H history  | -c clears ~/.novel

# Test / lint / docs — what CI and the pre-commit hook enforce
cargo test --locked --all-features --workspace --lib --tests --examples
cargo clippy --all-targets --all-features --workspace -- -D warnings   # -D warnings in CI
cargo fmt --all --check
RUSTDOCFLAGS="-D warnings" cargo doc --no-deps --document-private-items --all-features --workspace --examples

# A single test (the app crate `src/` has NO tests — they live only in the two sub-crates)
cargo test -p parse-book-source <test_name>
cargo test -p novel-tts <test_name>

# Crate-scoped examples
cargo run -p parse-book-source --example json     # needs a book-source test.json
cargo run -p novel-tts --example basic            # downloads model + plays real audio

# Docs site (docs/ — Astro + Starlight, formatted with Biome)
cd docs && pnpm install && pnpm dev               # pnpm build / preview / astro check
```

`lefthook.yaml` runs test → `clippy --fix --allow-dirty` (auto-stages fixes) → `cargo fmt` → `cargo doc` on pre-commit. Releases go through `./release.sh` (cargo-release + git-cliff changelog, tag `<crate>-v<version>`) and cargo-dist (`.github/workflows/trnovel-release.yml`, triggered by `trnovel-v*` tags).

## Architecture

### UI framework: ratatui-kit (`src/app`, `src/pages`, `src/components`, `src/hooks`)

`ratatui-kit` is a React-like layer over ratatui. Understanding its model is the key to this codebase:

- **Components** are `#[component] fn(props, hooks) -> impl Into<AnyElement>`; UI is composed with the `element!` macro.
- **Hooks** drive everything: `use_state` (local `State<T>`; `.read()`/`.write()`/`.set()`), `use_context` (global state), `use_future`/`use_async_effect` (async side effects), `use_events` (keyboard), `use_memo`. The repo adds custom hooks in `src/hooks` — notably `UseThemeConfig` (`use_theme_token.rs`) and `UseInitState` (`use_init_state.rs`, async-load state that debounces the loading spinner by 200ms).
- **Routing** is declared with the `routes!` macro in `src/app/mod.rs` and served by `RouterProvider`. Navigation uses `use_navigate()` + `navigate.push(path)` / `push_with_state(path, T)`; the target page reads typed state via `use_route_state::<T>()`. All pages render inside `src/app/layout.rs` (the `Layout` outlet).

### App boot & global state (`src/app/mod.rs`)

`App` mounts `use_future` to load all caches (History, BookSourceCache, TTSConfig, theme.json) off the UI path, showing a `Loading` spinner only if init exceeds ~200ms, then renders a **nested `ContextProvider` chain**: ThemeConfig → History → BookSourceCache → TTSConfig → NovelTTS → is_inputting → `RouterProvider`. Pages reach this state via `use_context`. Reordering the providers risks breaking descendant lookups.

Routes: `/home`, `/select-history`, `/select-file` → `/local-novel`, `/book-source` → `/select-books` → `/book-detail` → `/network-novel`, `/theme-setting`. The CLI subcommand (`src/lib.rs`, clap derive) selects the initial flow.

### Keyboard handling — no central keymap


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yexiyue/TRNovel](https://github.com/yexiyue/TRNovel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
