---
trigger: always_on
description: Validates god implementations by checking move generation against brute-force move enumeration. Verifies that:
---

# Santorini AI

Game engine for the board game Santorini using negamax search with NNUE evaluation.

### Workspace crates (`Cargo.toml`)
- **`santorini_core`** - Core game logic, gods, search, NNUE eval. The heart of the project.
- **`uci`** - UCI-like protocol interface for external UIs
- **`ui`** - Native analysis GUI built with egui
- **`wasm_app`** - WASM bindings for the web app
- **`battler`** - Runs automated games between engine configurations
- **`datagen`** - Generates training data for NNUE from self-play
- **`bullet_prep`** - Prepares NNUE training data in bullet format

### Other directories
- **`web_app/`** - TypeScript/Vite web frontend (deployed to GitHub Pages)
- **`python/`** - Data analysis scripts (`pull_data.py`, `read_matchups.py`, `ui.py`)
- **`game_data/`** - Generated game data files from datagen runs
- **`models/`** - NNUE model files (`.nnue` binary format)
- **`data/`** - Matchup configuration YAML files

### Key binaries in `santorini_core/src/bin/`
- `fuzzer.rs` - Fuzz testing for game logic consistency
- `visit_tester.rs` - Tests search visit counts across positions
- `tree_perf.rs` - Performance benchmarking for search tree traversal
- `post_process_model.rs` - Post-processes NNUE model files

### Battler binaries (`battler/src/bin/`)
- `run_matchups.rs` - Runs batch matchups between god pairs
- `compare_engines.rs` - Compares two engine configurations
- `faceoff.rs` - Runs a face-off between two specific configurations
- `single.rs` - Runs a single game between configurations
- `seed.rs` - Generates seed positions

## Core Game Model

### Board Representation (`board.rs`)
The 5x5 Santorini board is represented using bitboards (32-bit integers via `BitBoard`).

- **`BoardState`** - The core game state:
  - `current_player: Player` - Whose turn it is (`Player::One` or `Player::Two`)
  - `height_map: [BitBoard; 4]` - Four bitboard layers encoding building heights. `height_map[L-1]` has bit set for squares at height >= L (levels 1-4, where 4 = dome)
  - `workers: [BitBoard; 2]` - Worker positions per player (bitboard with bits set for worker squares)
  - `god_data: [u32; 2]` - God-specific state per player (e.g., Athena's "opponent can't climb" flag, Morpheus block count, Aeolus wind direction)
  - `hash: HashType` - Zobrist hash for transposition table
  - `height_lookup: [u8; 25]` - Cached height per square

- **`FullGameState`** - Combines `BoardState` with a `GodPair` (two `&'static GodPower` references). This is the primary state type used throughout the engine. Serializes as a FEN string.

- **`Square`** - Enum of 25 squares (A5..E1), stored as `u8`, indexed row-major from top-left.

- **`BitBoard`** - 32-bit bitboard. Bits 0-24 map to the 25 squares. Bits 25-29 reserved. Bits 30-31 encode winner state (in `height_map[0]`).
- Many square -> neighbor / push mappings (NEIGHBOR_MAP, INCLUSIVE_NEIGHBOR_MAP, WRAPPING_NEIGHBOR_MAP...) are precomputed.

### Move Representation
Moves are encoded as `GenericMove(u32)` - a 32-bit integer with bit-packed fields. Each god defines its own move struct (e.g., `MortalMove`, `ApolloMove`) that transmutes to/from `GenericMove`.

Typical fields packed into the u32:
- `move_from_position` (5 bits) - worker starting square
- `move_to_position` (5 bits) - worker destination square
- `build_position` (5 bits) - where to build
- Additional god-specific fields (e.g., swap square for Apollo)
- Bit 31 (`MOVE_IS_WINNING_MASK`) - flags the move as a win
- Bit 30 (`MOVE_IS_CHECK_MASK`) - flags the move as creating a check (threatening to win)

`ScoredMove` pairs a `GenericMove` with a `MoveScore` for ordering during search.

### FEN Format (`fen.rs`)
Board states serialize as FEN strings: `heights/current_player/god1:workers god2:workers`
Example: `10000 00000 00000 00000 00000/1/mortal:A1,A2 pan:E4,E5`

## Implementing Gods

### Architecture Overview
Gods are implemented as **static function pointers** assembled into a `GodPower` struct, not as trait objects. The `GodPower` struct holds ~20 function pointers covering move generation, move application, placement, and serialization.

### Pattern for Adding a New God

1. **Create a new file** `santorini_core/src/gods/your_god.rs`

2. **Define a move struct** that wraps `MoveData` (u32):
```rust
#[derive(Copy, Clone, PartialEq, Eq)]
pub struct YourGodMove(pub MoveData);
```
Pack move fields (from_square, to_square, build_square, etc.) into the u32 using bit shifts. Use `POSITION_WIDTH` (5 bits) per square.

3. **Implement `GodMove` trait** for your move struct:
   - `move_to_actions()` - Converts the compact move into a `Vec<FullAction>` (list of `PartialAction` steps for the UI)
   - `make_move()` - Applies the move to a `BoardState` mutably. Call `board.worker_xor()` to move workers, `board.build_up()` to build, `board.set_winner()` for wins
   - `get_blocker_board()` - Returns a `BitBoard` of squares this move interacts with during a winning move to help with blocking checks during search (behaviour when not winning is undefined).
   - `get_history_idx()` - Returns a unique index for the move (used for history heuristic in search)

4. **Implement `Into<GenericMove>` and `From<GenericMove>`** using `unsafe { std::mem::transmute(self) }`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JPricey/santorini-ai](https://github.com/JPricey/santorini-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
