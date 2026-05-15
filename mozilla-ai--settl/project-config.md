---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
cargo build                              # Debug build
cargo build --release                    # Release build (binary: target/release/settl)
cargo test                               # All tests
cargo test game::rules                   # Tests in a specific module
cargo test test_name                     # Single test by name
cargo run                                # Launch TUI (title screen -> menus -> game)
```

The binary boots into a TUI (title screen -> main menu -> game setup). AI players run locally via llamafile by default, no API keys needed.

Debug logging writes to `~/.settl/debug.log`. Enabled automatically in debug builds, off in release. Override with env var:
```bash
cargo run                                # Debug build: logging on automatically
SETTL_DEBUG=0 cargo run                  # Debug build: force logging off
SETTL_DEBUG=1 cargo run --release        # Release build: force logging on
```
Use `log::debug!()` / `log::info!()` etc. from any module.

## Architecture

**settl** is a terminal hex settlement game where LLMs play via tool/function calling. The codebase has four modules:

### `game/` -- Core engine (stateless rules + stateful orchestrator)
- **`board.rs`** -- Hex grid using axial coordinates `(q, r)`. Vertices and edges are expressed as `(HexCoord, Direction)` pairs. Only canonical edge directions (NE, E, SE) are stored; opposites resolve to the neighbor's canonical form.
- **`state.rs`** -- `GameState` holds the full mutable game: board, per-player resources/cards (`PlayerState`), buildings, roads, robber position, dev card deck, longest road/largest army tracking. `GamePhase` enum drives the state machine (Setup -> Playing -> Discarding -> PlacingRobber -> Stealing -> GameOver).
- **`rules.rs`** -- Pure validation functions. Given a `GameState`, returns legal moves. Enforces distance rule, connectivity, resource costs, dev card logic, longest road calculation. Largest file (~2400 lines).
- **`actions.rs`** -- Action and dev card type definitions (`DevCard`, `PlayerAction`, etc.) used across the engine.
- **`event.rs`** -- `GameEvent` enum for all discrete game actions; `format_event()` renders them as human-readable text for LLM context.
- **`orchestrator.rs`** -- Drives the game loop. Calls `Player` trait methods at decision points, applies actions through the rules engine, tracks events for LLM context, sends UI updates via `mpsc` channel. Runs the setup snake-draft and main turn loop.
- **`dice.rs`** -- Dice rolling and resource distribution per hex/number.
- **`save.rs`** -- Auto-save and resume. Saves game state to `~/.settl/saves/autosave.json` after each turn; main menu shows "Continue" when a save exists.

### `player/` -- Player abstraction (async trait)
- **`mod.rs`** -- `Player` trait with async methods: `choose_action`, `choose_settlement`, `choose_road`, `choose_resource`, `respond_to_trade`, etc. Each returns `(choice, reasoning_string)`.
- **`anthropic_client.rs`** -- Thin HTTP client for the Anthropic Messages API (`/v1/messages`). Works with both local llamafile/llama.cpp and real Anthropic API. Includes llamafile-specific extensions (`id_slot`, `cache_prompt`) for KV cache slot management.
- **`llm_player.rs`** -- `LlmPlayer` talks to the Anthropic Messages API via `anthropic_client`. Maintains per-player conversation history for KV cache efficiency. Defines JSON-schema tools (`choose_index`, `choose_resource`, `choose_discard`, `propose_trade`, `respond_to_trade`) for structured responses. Retries up to 2x on parse failure, falls back to random.
- **`random.rs`** -- `RandomPlayer` for testing and `--demo` mode.
- **`human.rs`** -- `HumanPlayer` for raw stdin input (non-TUI).
- **`tui_human.rs`** -- `TuiHumanPlayer` for TUI mode; communicates with the UI via channels to show a selection overlay.
- **`prompt.rs`** -- Serializes board/state into text for LLM context.
- **`personality.rs`** -- Loads TOML personality configs (aggression/cooperation scores, style text, catchphrases) and injects into system prompts. Built-in personalities: Balanced Strategist, The Merchant, The Grudge Holder, The Architect, The Wild Card. Custom ones go in `personalities/*.toml`.

### `trading/` -- Trade negotiation
- **`negotiation.rs`** -- Multi-round trade protocol: propose -> respond (accept/reject/counter) -> execute.
- **`offers.rs`** -- Trade validation (both sides have resources, no self-trades) and `trade_value_heuristic()` scoring.

### `ui/` -- TUI (ratatui + crossterm)
- Async game engine runs in a background tokio task; TUI runs on the main thread.
- Communication via `mpsc::unbounded_channel` sending `StateUpdate` events.
- `board_view.rs` renders the hex board, `chat_panel.rs` shows AI reasoning, `resource_bar.rs` shows player stats, `game_log.rs` is scrollable event history.
- `layout.rs` handles responsive terminal layout and panel sizing.
- `menu.rs` renders menu screens (title, game setup, etc.).

## Key Design Decisions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mozilla-ai/settl](https://github.com/mozilla-ai/settl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
