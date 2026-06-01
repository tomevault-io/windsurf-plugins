---
trigger: always_on
description: Handles fiat-fiat, fiat-crypto, crypto-fiat, crypto-crypto uniformly.
---

# glint — Terminal Dashboard

## What This Is

glint is a keyboard-driven terminal dashboard (Rust + Ratatui) that
displays clock, weather, calendar, news, email, stocks, forex (with
crypto), system resources, an image gallery, and a vim-flavoured notes
pad in a configurable grid layout. Cells can be single widgets or
**stacks** that rotate between multiple widgets. Pre-launch v0.2;
shipping under GPL v3-or-later (see `LICENSE` + `CONTRIBUTING.md`).

For user-facing setup, see `README.md` and `INSTRUCTIONS.md`.

## Tech Stack

- **Rust 2021 edition** with Tokio async runtime
- **Ratatui 0.28+** for TUI rendering (crossterm backend)
- **reqwest** for HTTP (Yahoo Finance, Open-Meteo, Google APIs, Microsoft
  Graph, RSS feeds, IMAP via `imap` crate, Anthropic + OpenAI APIs).
  A single process-wide client lives in `src/http.rs::shared()`.
- **serde + toml** for config; **serde_json** for API responses
- **chrono + chrono-tz** for timezone-aware date/time
- **strsim** for fuzzy command matching
- **ratatui-image + image** for the Gallery widget's inline rendering
- **imap + native-tls + mail-parser** for the Email widget's IMAP path
- **readability + url** for the News widget's optional article-body
  extraction (LLM summaries)
- **sysinfo** for the Resources widget's CPU / memory / process info
- **lru + sha2** for in-memory caches keyed by content hash
- Config format: **TOML** (not JSON, not YAML)

## Project Structure

```
src/
├── main.rs                  # Entry point, CLI parsing (clap), runtime setup
├── app.rs                   # App state, focus model, command dispatch, live-reload
├── event.rs                 # Event loop: crossterm input + tick + config watch
├── http.rs                  # Process-wide shared reqwest::Client (`shared()`)
├── clipboard.rs             # OSC-52 clipboard write helper
├── geolocation.rs           # IP / name-based location lookup (weather, clock)
├── runtime_state.rs         # Per-process state persisted to ~/.config/glint/.runtime_state.toml
├── cache/
│   └── mod.rs               # Persistent on-disk cache (JSON + bytes). See Cache Layer.
├── config/
│   ├── mod.rs               # Per-widget TOML loader; XDG paths; --init seeds
│   ├── layout.rs            # Grid layout parsing and resolution
│   ├── types.rs             # Top-level Config struct
│   └── watcher.rs           # `notify`-based config file watcher
├── auth/
│   ├── mod.rs               # credentials_dir helper
│   ├── registry.rs          # AuthProvider registry — self-describing entries
│   │                        #   carry display_name, credentials_spec, post_auth_refresh
│   ├── loopback.rs          # localhost OAuth redirect listener
│   ├── google/              # Google OAuth client + token store
│   └── microsoft/           # Microsoft OAuth client + token store (PKCE)
├── widgets/
│   ├── mod.rs               # Widget trait, WidgetCtx, WidgetManager
│   ├── registry.rs          # WIDGETS table — add a descriptor to register
│   ├── stack.rs             # StackWidget — composite cell holding N child widgets
│   │                        #   with tab-strip rotation via . / ,
│   ├── clock/, weather/, calendar/, news/, stocks/, forex/, email/,
│   │       resources/, gallery/, notes/
│   │   └── mod.rs + helpers — each is a self-contained widget module
│   │      with `pub const KIND` and `pub fn build(&WidgetCtx)`.
├── llm/
│   ├── mod.rs               # LlmProvider trait, LlmProviderDef registry,
│   │                        #   LlmRequest/LlmResponse types
│   ├── anthropic.rs         # AnthropicProvider (Messages API)
│   ├── openai.rs            # OpenAiProvider (Chat Completions API)
│   ├── rate_limiter.rs      # Token-bucket request budget tracking
│   └── cache.rs             # In-memory LRU response cache (L1), TTL-evicted
├── theme/
│   └── mod.rs               # Color scheme loader, per-widget overrides
├── ui/
│   ├── mod.rs               # Top-level renderer, unified title row helper,
│   │                        #   command bar, status bar wiring
│   ├── status_bar.rs        # Bottom bar: theme, command feedback, clock
│   ├── help.rs              # "?" help overlay (sources keybindings from widgets)
│   └── big_digits.rs        # Block-digit renderer for clock
└── wizard/
    ├── mod.rs               # --setup TUI wizard entry
    ├── app.rs               # Wizard event loop + page dispatcher
    ├── descriptor.rs        # WizardField / WizardFieldKind types
    ├── flow.rs              # Page ordering (Welcome → Global → Layout → …)
    ├── hydrate.rs           # Seed wizard state from existing on-disk config
    ├── finalize.rs          # Write final TOMLs at "Complete and Save"
    ├── state.rs             # In-memory wizard state buffer
    ├── storage.rs           # Resume-buffer persistence
    ├── style.rs, toml_merge.rs
    └── pages/               # One renderer per page (welcome, global, layout,
                             #   assign, widget, oauth_setup, assign_stack,
                             #   confirm, preview)
```

## Core Traits and Registries

Five extension points define the architecture — know these before
touching anything material.

### Widget (src/widgets/mod.rs)
The runtime contract for everything that lives in a grid cell. The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ntrospect0/glint](https://github.com/ntrospect0/glint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
