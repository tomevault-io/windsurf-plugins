---
trigger: always_on
description: When file becomes bigger then 1.5k lines suggest refactoring
---

# User instructions, editing not allowed
When file becomes bigger then 1.5k lines suggest refactoring
Use love2d and lua best coding practicies
DO NOT use goto, it breaks the web build
Document what you have done
Keep your visuals and logic separate.
# End of user instructions.

## Modules

| File | Role |
|---|---|
| `main.lua` | Entry point: LOVE callbacks, window setup, asset loading, screen registration |
| `coin_sort.lua` | Coin Sort mode logic. Coins are `{number=N}` objects (1-50). |
| `coin_sort_screen.lua` | Coin Sort mode screen (UI, input, drawing) |
| `game_over_screen.lua` | Post-run stats: score, resource summary, Continue to Arena button |
| `arena.lua` | Merge Arena logic: 7x8 grid, boxes/sealed/items, dispenser, stash, generators |
| `arena_chains.lua` | 12 item chains with colors, items, generator specs, drop tables (data only) |
| `arena_orders.lua` | Level-based orders (10 levels, static data), completion, rewards (data only) |
| `arena_screen.lua` | Merge Arena screen: grid, dispenser, stash, orders, tutorial, drag-and-drop |
| `screens.lua` | Screen manager with mode selection |
| `animation.lua` | Dual-track animation: pick/place + merge/deal run independently, 1.5x speed |
| `particles.lua` | Chunky bouncy coin fragments with custom physics (weighty, impactful feel) |
| `graphics.lua` | Game rendering: coins, boxes, background (NOT UI buttons) |
| `input.lua` | Hit testing and coordinate conversion |
| `layout.lua` | Centralized layout: 1080x2400 virtual canvas, scaling, grid metrics |
| `resources.lua` | Fuel/Stars resource system (data only, no drawing) |
| `bags.lua` | Coin bag inventory + free bag timer (data only, no drawing) |
| `tab_bar.lua` | Bottom tab bar UI component for screen switching, badge counts |
| `commissions.lua` | Commission system for Coin Sort: persistent Forge/Harvest goals with Bag+Star rewards, manual collect, batch refresh (data only) |
| `drops.lua` | Variable drop system: cross-mode rewards (Chest, FuelSurge, StarBurst, GenToken, Hammer, AutoSort, BagBundle, DoubleMerge) |
| `powerups.lua` | Consumable power-ups: Auto Sort, Hammer (data only) |
| `progression.lua` | Unlock/achievement system with file persistence (`progression.dat`) |
| `coin_utils.lua` | Coin Sort helpers: 5-color cycling, shard mapping |
| `sound.lua` | Sound loading, playback, toggle state |
| `utils.lua` | `each_coin()` iterator and debugger setup |
| `conf.lua` | LOVE window config (resizable, HiDPI) |
| `skill_tree.lua` | PoE2-style skill tree: 30 nodes, query API, migration (data only, no drawing) |
| `skill_tree_screen.lua` | Skill tree full-screen UI: pannable node graph, detail panel, unlock interaction |
| `effects.lua` | Screen-level visual effects: fly-to-bar icons, overlay flash, celebration burst (pre-allocated pools) |
| `popups.lua` | Popup queue system: toast/card/celebration tiers, FIFO queue, rendering (UI overlay) |
| `tutorial.lua` | Placeholder for future tutorial |
| `arena_icons.lua` | Item icon sprite loading and rendering for Merge Arena (visual only, no game logic) |
| `yandex.lua` | Yandex Games SDK bridge: ads (interstitial, rewarded, banner) via Emscripten FFI, no-ops on non-web |

## Key Patterns

- **No goto** — use `repeat/until` loops for retries. Goto breaks the web build.
- **Logic/visual separation** — data modules (`resources`, `bags`, `coin_sort`, `powerups`) have zero drawing code; screen modules handle all rendering.
- **Module exports** — each module returns a table of public functions.
- **Iterator** — `utils.each_coin(boxes)` for coin traversal.
- **Immediate-mode UI** — buttons drawn as rectangles, hit-tested on mouse click.

## Rendering

1080x2400 virtual canvas (portrait) with letterboxing. Screen-to-game coord conversion via `ox`, `oy`, `scale` in `main.lua`. Coins are always rendered as tinted `ball.png`.

## Screen System

Each screen is a table with optional methods: `enter()`, `exit()`, `update(dt)`, `draw()`, `mousepressed(x, y, button)`, `keypressed(key, scancode, isrepeat)`.

**Active Screens:** `coin_sort` (Coin Sort, default start), `arena` (Merge Arena), `game_over`, `skill_tree` (Skill Tree)
**Dormant Screens:** `mode_select` — kept in code but not registered.

**Adding a new screen:**
1. Create `my_screen.lua` with screen table and methods
2. Add `my_screen.init(assets)` to receive shared assets
3. In `main.lua`, require, init, and register with `screens.register()`

## Two-Screen Loop (Coin Sort + Merge Arena)

```
[App Start] -> [coin_sort] <--tab bar--> [arena]
                   |                        ^
             [all boxes full,               |
              no merges possible]           |
                   |                        |
                   v                        |
             [game_over]                    |
                   |                        |
             ["Continue to Arena"] ---------+
```

1. `coin_sort_screen.enter()` uses fixed 3×5 grid (15 boxes, 10 slots each), inits game
2. Deal coins from bags (limited). Merge coins to earn Fuel/Stars.
3. Switch to Arena via tab bar. Generators cost 1 Fuel per tap to produce items.
4. Completing arena orders rewards XP + items (to dispenser queue).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gr13nka/Coins-Boxes](https://github.com/gr13nka/Coins-Boxes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
