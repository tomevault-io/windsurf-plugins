---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Wordle Battle is a real-time competitive Wordle game built with Phoenix LiveView. Players compete simultaneously to guess the most 5-letter words within a timed session, with scoring based on speed and accuracy.

## Essential Commands

### Development
- `mix setup` - Install dependencies and build assets (first-time setup)
- `mix phx.server` - Start Phoenix server (visit http://localhost:4000)
- `iex -S mix phx.server` - Start Phoenix server in interactive Elixir shell
- `mix precommit` - Run full pre-commit checks (compile with warnings as errors, format, test)

### Testing
- `mix test` - Run all tests
- `mix test test/path/to/test.exs` - Run specific test file
- `mix test --failed` - Run only previously failed tests

### Assets
- `mix assets.build` - Build Tailwind + esbuild assets
- `mix assets.deploy` - Build and minify assets for production

## Architecture

### Game State Management (GenServer + LiveView)

The game uses a **GenServer-based architecture** with Phoenix LiveView for real-time UI:

1. **WordleServer (GenServer)**: Each game session runs as a supervised GenServer process
   - Manages game state, phase transitions (:lobby → :playing → :game_over)
   - Handles word assignment, guess validation, scoring
   - Broadcasts state updates via PubSub
   - Persists state across player disconnections

2. **WordleLive (LiveView)**: Real-time UI for players
   - Subscribes to PubSub topic for session updates
   - Renders game grid, virtual keyboard, leaderboard
   - Handles user interactions (guesses, ready state)
   - Manages player reconnection via localStorage user_id

3. **Dynamic Supervision**: Sessions registered via Registry for efficient lookup
   - `Registry.lookup/2` for finding session processes
   - `DynamicSupervisor` for fault-tolerant session management

4. **PubSub Broadcasting**: Real-time updates to all session participants
   - Timer updates every second
   - Score changes after each guess
   - Player connection status changes

### Word Management

- **Dictionary Loading**: Answer words (~2,300) and valid guesses (~12,000) loaded at app startup in `application.ex`
- **Module**: `WordleBattle.Dictionary` - provides `answer_words/0`, `valid_guesses/0`, `random_word/1`
- **Per-Session Tracking**: `used_words` list prevents word repetition within a session
- **Files**: Located in `priv/dictionary/answer_words.txt` and `priv/dictionary/valid_guesses.txt`

### Player Persistence

- **UUID Storage**: Each player gets persistent UUID in browser localStorage
- **Automatic Reconnection**: Players rejoin session on page refresh via stored user_id
- **Graceful Degradation**: Disconnected players' progress preserved; game continues for others
- **Cleanup Workers**: Automated cleanup of disconnected players (10 min) and inactive sessions (1 hour)

## Key Game Mechanics

### Scoring System
- **Base**: 1 point per correctly guessed word
- **Bonuses**:
  - 1 attempt: +5 (total 6 pts)
  - 2 attempts: +3 (total 4 pts)
  - 3 attempts: +2 (total 3 pts)
  - 4-6 attempts: +1 (total 2 pts)
- **Failed word**: 0 points

### State Machine
```elixir
:lobby -> :playing  # All players ready → start_game
:playing -> :game_over  # Timer expires → calculate winners
:game_over -> :lobby  # start_new_game → reset with preserved players
```

### Guess Validation Algorithm
Two-pass algorithm to handle duplicate letters correctly:
1. First pass: Mark exact matches (green)
2. Second pass: Mark present letters (yellow) from remaining pool
3. Mark remaining as wrong (gray)

## Project-Specific Guidelines

### Phoenix v1.8 Patterns
- Use `<Layouts.app>` wrapper in all LiveView templates (already aliased in `wordle_battle_web.ex`)
- All forms use `to_form/2` in LiveView, access via `@form[:field]` in templates
- Use `<.link navigate={}>` and `<.link patch={}>`, not deprecated `live_redirect`/`live_patch`
- Icons via `<.icon name="hero-x-mark" />` component from core_components.ex

### LiveView Streams for Collections
Use streams for player lists, word history, and leaderboards to prevent memory issues:
```elixir
# In LiveView
stream(socket, :players, [new_player])  # append
stream(socket, :players, players, reset: true)  # replace all

# In template (must set phx-update="stream" on parent)
<div id="players" phx-update="stream">
  <div :for={{id, player} <- @streams.players} id={id}>
    {player.nickname}
  </div>
</div>
```

### Testing with Phoenix.LiveViewTest
- Always add unique DOM IDs to elements (`<.form for={@form} id="guess-form">`)
- Reference these IDs in tests: `assert has_element?(view, "#guess-form")`
- Use `render_submit/2` and `render_change/2` for form testing
- Test outcomes, not implementation details
- Use `LazyHTML` for debugging complex selectors

### Word Validation
- All words normalized to uppercase
- Validate guesses against `Dictionary.valid_guesses/0` MapSet (O(1) lookup)
- Assign only from `Dictionary.answer_words/0` list (~2,300 common words)

### Timer Implementation
Use `:timer.send_interval/2` in GenServer for countdown:
- Broadcast time updates via PubSub every second

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ElixirMentor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
