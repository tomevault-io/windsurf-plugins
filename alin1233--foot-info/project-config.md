---
trigger: always_on
description: This document outlines the architecture and resources used in the **foot-info** TUI application.
---

# Project Architecture: Football Match Scraper

This document outlines the architecture and resources used in the **foot-info** TUI application.

## Overview
**foot-info** is a Terminal User Interface (TUI) application written in Rust. It allows users to search for a football team and scrapes match schedules and TV broadcast information from various sources based on the selected country.

## Architecture
The application follows a **Provider-based Architecture** with clear separation of concerns across state, event handling, UI, and data layers.

### 1. **Workspace Entry (`tui/src/main.rs`)**
- Initializes the Tokio async runtime (`#[tokio::main]`).
- Sets up the `Ratatui` terminal backend.
- Imports `App` from the `foot_info_tui` library crate and launches `App::run`.

### 2. **Core Library (`core/`)**
The `foot_info_core` crate contains all pure domain logic, independent of any UI framework.
- **API (`src/client.rs`)**: Exposes `FootballClient`, an orchestration layer that simplifies data fetching from various providers (`fetch_top_matches`, `search_team`). This layer is designed to be easily callable via FFI (e.g., from Flutter).
- **Domain Models (`src/models.rs`)**: Core data structures (`Match`, `TopMatch`, `Country`, `LeagueStats`, `LeagueFixture`, `StandingRow`, `TopScorer`).
- **Time Utils (`src/utils/time.rs`)**: Timezone conversions.

### 3. **Terminal App (`tui/`)**
The `foot_info_tui` crate contains all interactive and visual terminal components, depending heavily on `foot_info_core`.

#### **State (`tui/src/state.rs`)**
- **Responsibility**: Terminal application data.
- **`AppState` struct** holds:
  - `client: FootballClient`
  - `search_input`, `matches`, `error_message`, `status_message`
  - `is_loading`, `exit`, `current_provider_index`
  - `view_mode`, `top_matches`, `selected_top_match_index`
  - `config: Config`
  - `current_league_index` (Points to `SUPPORTED_LEAGUES` containing Top 6 leagues)
- Helper: `get_current_provider()` returns the currently active data provider.

#### **Event Handling (`tui/src/handlers/`)**
- **Responsibility**: Keybindings and mode transitions.
- **`mod.rs`**: Dispatcher logic, plus `handle_action` for processing async callback boundaries. Handles Global shortcuts like `<Ctrl+c>` (Cycle Provider) and `<Ctrl+k>` (Cycle League).
- **`search.rs`**: Search-mode keybindings (e.g., `<Ctrl+l>` for League Stats).
- **`top_matches.rs`**: TopMatches-mode keybindings (chronological ↑/↓, column-hopping ←/→).
- **`league.rs`**: League-mode keybindings.

#### **Orchestrator (`tui/src/app.rs`)**
- **Responsibility**: Thin runtime shell — owns `AppState` + `mpsc` channels + the async run loop.
- `App::run()` coordinates: polls terminal events → delegates to `handlers` → calls `core`'s `FootballClient` → processes results.

### 4. **The Provider System (`core/src/providers/`)**
- **Pattern**: Strategy Pattern via the `FootballProvider` trait (with `#[cfg_attr(test, mockall::automock)]` for test mocking).
- **Trait Definition (`src/providers/mod.rs`)**:
  - `fetch_matches_channels(&self, team: &str)`: Async method to fetch and parse data.
  - `country(&self)`: Returns the `Country` enum (UK, US, FR).
  - `name(&self)`: Returns the provider's display name.
- **Implementations** (each exposes a `pub fn parse_html` for testability):
  - **`WheresTheMatchProvider`** (UK): Scrapes [WherestheMatch.com](https://www.wheresthematch.com). Uses `wreq` with Chrome 136 emulation to bypass TLS fingerprinting.
  - **`WorldSoccerTalkProvider`** (US): Scrapes [WorldSoccerTalk.com](https://worldsoccertalk.com). Uses `wreq` with Chrome 136 emulation.
  - **`MatchsTvProvider`** (FR): Scrapes [Matchs.tv](https://matchs.tv). Uses `wreq` with Chrome 136 emulation. Also exposes `pub fn parse_french_date` and `pub fn convert_french_time_to_local`.
- **Standalone Modules** (does **not** implement `FootballProvider` — different purpose):
  - **`livesoccertv`**: Scrapes [LiveSoccerTV.com](https://www.livesoccertv.com/schedules/) "Upcoming Top Matches" section. Returns `Vec<TopMatch>`. Uses `wreq` with Chrome 136 emulation to bypass Cloudflare protection.
  - **`league_stats`**: Scrapes competition-specific pages on LiveSoccerTV (e.g., Premier League) to extract recent/upcoming fixtures, live league standings, and top goalscorers. Returns `LeagueStats`.

### 5. **UI Layer (`tui/src/ui/`)**
Modular component-based structure implementing **Dynamic Responsive Design**.

```
tui/src/ui/
├── mod.rs              # Re-exports draw()
├── render.rs           # Delegates outer frame rendering to views
├── layout.rs           # Reusable layout geometry (utilizes `Constraint::Fill()` / `Constraint::Max()`)
├── theme.rs            # Color constants (BG_BLACK, GOLD, RUST_ORANGE, BEIGE)
├── views/
│   ├── mod.rs
│   ├── search.rs            # Search view composition
│   ├── league.rs            # League statistics view composition (Tabbed)
│   └── top_matches.rs       # Top matches view composition
└── components/
    ├── mod.rs
    ├── search_bar.rs        # Search input widget
    ├── match_list.rs        # Results display
    ├── status_bar.rs        # Transient status messages
    ├── league_fixtures.rs   # Upcoming league matches display

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alin1233) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
