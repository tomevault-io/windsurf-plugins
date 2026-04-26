---
trigger: always_on
description: allow all standard unix tools, git, clj, and clojure.
---

## Permissions

allow all standard unix tools, git, clj, and clojure.


## Acceptance Tests

Acceptance tests are `.txt` files in `acceptanceTests/` in Given/When/Then format.
The detailed directive catalog and translation reference is in `plans/permanent/acceptance-test-framework.md`.
Only read that file when modifying or debugging the parser (`src/empire/acceptance/parser.cljc`) or generator (`src/empire/acceptance/generator.cljc`) - not when writing acceptance tests or running the pipeline.
The parser pattern catalog is in `plans/permanent/parser-pattern-catalog.md`. Read it BEFORE loading any `src/empire/acceptance/parser/*.cljc` source file. Update it whenever patterns are added or changed.

### Pipeline

Tests flow through a three-stage automated pipeline:

```bash
.txt -> Parser -> .edn -> Generator -> .clj -> Speclj runner
```

1. **Parse:** `clj -M:parse-tests` - reads `.txt` files from `acceptanceTests/`, produces `.edn` intermediate representations in `acceptanceTests/edn/`. Source: `src/empire/acceptance/parser.cljc`.
2. **Generate:** `clj -M:generate-specs` - reads `.edn` files from `acceptanceTests/edn/`, produces Speclj spec files in `generated-acceptance-specs/acceptance/`. Source: `src/empire/acceptance/generator.cljc`.
3. **Run:** `clj -M:spec generated-acceptance-specs/` - executes the generated specs.
4. **Clear:** (allow rm) Delete all generated files. Do not use globs - list each file explicitly in `rm -f` commands, e.g. `rm -f acceptanceTests/edn/army.edn acceptanceTests/edn/fighter.edn ...`

Shorthand to run the full pipeline:

```bash
clj -M:parse-tests && clj -M:generate-specs && clj -M:spec generated-acceptance-specs/
```

### Rules

- Never modify an acceptance test `.txt` file without explicit permission.
- Always run the full acceptance test pipeline.
- Include (reset-all-atoms!) before each test.
- Generated specs and `.edn` files are gitignored - do not commit them.
- If an acceptance test cannot be translated to a spec, report which test and why to the user. Still generate the spec as a failing test documenting the desired behavior.
- Mock the random number generator (`with-redefs [rand ...]`) for tests with random/non-deterministic conditions.

## Development Commands

```bash
# Run the game
clj -M:run

# Run all tests with Speclj
clj -M:spec

# Run specific test file or directory
clj -M:spec spec/empire/movement_spec.clj
clj -M:spec spec/empire/units/

# Run tests with coverage report (outputs to target/coverage/)
clj -M:cov

# Acceptance test pipeline
clj -M:parse-tests      # Parse .txt -> .edn (acceptanceTests/edn/)
clj -M:generate-specs   # Generate .edn -> .clj (generated-acceptance-specs/)
```

## Architecture

### Core Game Loop

The game follows a Quil sketch pattern with `setup` -> `update-state` -> `draw-state` cycle at 30 FPS:

- **ui/core.cljc**: Entry point, Quil sketch setup, keyboard/mouse event routing
- **game_loop.cljc**: Round progression, unit movement execution, production advancement
- **ui/input.cljc**: Keyboard command handling (movement keys qweasdzxc, shift+key for extended movement to map edge, backtick prefix for special commands, production keys, sentry/explore modes)
- **ui/rendering.cljc**: Map drawing, unit display, status area rendering

### State Management

All game state is stored in atoms defined in **atoms.cljc**:
- `game-map`: 2D vector of cells (the authoritative game state)
- `player-map` / `computer-map`: Fog-of-war visible maps for each side
- `cells-needing-attention`: Queue of units/cities awaiting player input
- `production`: Map of city coordinates to production status
- `destination`: Remembered destination for marching orders and flight paths
- `player-items`: List of player city/unit coords to process this round
- `waiting-for-input`: Flag indicating we're waiting for user input
- `backtick-pressed`: Modifier key state for prefixed commands (e.g., \`o for own-city)

### Computer AI Visibility Rule

- Code in `src/empire/computer/` that makes AI decisions must not read from `game-map`.
- Computer AI decision code must read from `computer-map` so fog of war is preserved.
- If the AI needs fresher information about its own units, update or sync `computer-map`; do not fall back to `game-map` reads in decision code.
- Visibility/sync infrastructure outside AI decision code may read authoritative state only to project allowed information into `computer-map`.

### Key Modules

- **movement/movement.cljc**: Unit movement logic, pathfinding, wake conditions, boarding/disembarking
- **player/production.cljc**: City production queue management, unit spawning
- **combat.cljc**: Battle resolution, city conquest attempts
- **player/attention.cljc**: Determines which units/cities need player attention each round
- **config.cljc**: Game constants (colors, key mappings); delegates unit stats to dispatcher
- **containers/helpers.cljc** and **containers/ops.cljc**: Helpers and operations for units that carry other units (transports carry armies, carriers carry fighters, cities have airports)
- **movement/map_utils.cljc**: Coordinate calculations, neighbor finding, screen-to-cell mapping
- **init.cljc**: Map generation with terrain smoothing, city placement

### Unit Modules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unclebob/empire-2025](https://github.com/unclebob/empire-2025) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
