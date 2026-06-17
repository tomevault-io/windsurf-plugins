---
trigger: always_on
description: **ScoreHub** is an Android board game score tracking application, built in Kotlin using a View-based architecture (not Compose). It currently supports Cactus, Cribbage, Escoba, Farkle, Skyjo, Tarot, Wingspan, Yahtzee, and Akropolis.
---

# ScoreHub — Instructions for Claude

## Project overview

**ScoreHub** is an Android board game score tracking application, built in Kotlin using a View-based architecture (not Compose). It currently supports Cactus, Cribbage, Escoba, Farkle, Skyjo, Tarot, Wingspan, Yahtzee, and Akropolis.

- **Package** : `com.github.trivialloop.scorehub`
- **Min SDK** : 24 (Android 7.0)
- **Target SDK** : 34
- **Language** : Kotlin
- **UI** : XML Views + ViewBinding (no Jetpack Compose)
- **DB** : Room (SQLite)
- **License** : GPL v3

---

## Architecture

```
app/src/main/java/com/github/trivialloop/scorehub/
├── MainActivity.kt                  # Home screen, game selection
├── GeneralStatsActivity.kt          # Cross-game general statistics
├── SettingsActivity.kt              # Language + theme
├── ScoreHubApplication.kt           # Application class (theme init)
├── data/
│   ├── AppDatabase.kt               # Room database singleton
│   ├── Player.kt                    # Player entity (id, name, color ARGB, createdAt)
│   ├── GameResult.kt                # Result entity (gameType, playerId, score, isWinner, isDraw)
│   ├── PlayerDao.kt                 # Player CRUD
│   ├── GameResultDao.kt             # Stats + top20 queries
│   └── PlayersColors.kt             # Available color palette
├── games/
│   ├── akropolis/
│   ├── cactus/
│   ├── cribbage/
│   ├── escoba/
│   ├── farkle/
│   ├── skyjo/
│   ├── tarot/
│   ├── wingspan/
│   └── yahtzee/
└── utils/
    ├── LocaleHelper.kt              # Runtime language switching
    ├── ScoreColorHelper.kt          # Shared utility: scoreColorRole()
    └── ThemeHelper.kt               # Light / Dark / System
```

---

## Important rules

### General
- All variable names, function names, and comments must be in **English**.

### Database
- **Never modify** `AppDatabase` without creating a **Room migration** (`addMigrations(...)` in the builder).
- Current version is **`version = 1`** — any column or table addition must increment this number and provide the SQL migration script.
- `GameResult.gameType` is a plain string (`"yahtzee"`, `"skyjo"`, etc., not an enum).

---

## UI / Theme

- Theme is `Theme.MaterialComponents.DayNight.NoActionBar`.
- Always use **semantic colors** (`?attr/colorPrimary`, `?android:attr/colorBackground`, etc.) rather than hardcoded values, so dark mode works automatically.
- Adaptive colors (game cells) are defined in `res/values/colors.xml` **and** `res/values-night/colors.xml`.

### Unified cell color system (all games)

All game grids share a single set of semantic color names. **Never add per-game color names.**

| Color name | Light | Dark | Usage |
|---|---|---|---|
| `cell_editable_bg` | `#FFF8E1` | `#454545` | Empty cell that can be filled right now |
| `cell_editable_filled_bg` | `#FFFFFF` | `#2C2C2C` | Cell already filled but still editable |
| `cell_locked_bg` | `#F0F0F0` | `#1A1A1A` | Cell filled and locked (no longer editable) |
| `cell_never_bg` | `#BDBDBD` | `#000000` | Cell that can never be filled (e.g. first-player crib column) |
| `score_cell_background` | `#FFFFFF` | `#2C2C2C` | Neutral/read-only score cell background |
| `score_cell_text` | `#000000` | `#FFFFFF` | Neutral score text |
| `score_text_best` | `#4CAF50` | `#4CAF50` | Best score text (green) |
| `score_text_worst` | `#F44336` | `#F44336` | Worst score text (red) |
| `cell_border` | `#CCCCCC` | `#555555` | Cell stroke/border |
| `header_cell_background` | `#D3D3D3` | `#3A3A3A` | Header/label cell background |
| `header_cell_text` | `#000000` | `#FFFFFF` | Header/label cell text |

Legacy aliases (e.g. `skyjo_score_green`, `cactus_cell_border`) remain in `colors.xml` as `@color/` references to the canonical names above. Do not use legacy names in new code.

### Score coloring rules (uniform across all games)

Per row (manche / category row / etc.):
- **All values identical** → all neutral.
- **Minimum value** → `score_text_best` (green), **bold**.
- **Maximum value** → `score_text_worst` (red), **bold**.
- **Everything else** → `score_cell_text` (neutral).

For Skyjo (lower = better): minimum → red, maximum → green (use `ScoreColorHelper.scoreColorRole(lowerIsBetter = true)`).

Use `ScoreColorHelper.scoreColorRole(value, allValues)` from `utils/ScoreColorHelper.kt` for all score color decisions.

### Editable cell visual convention

- **Pencil prefix**: when a cell is already filled but still editable, prefix the dialog title with a ✏ emoji.
- **Background color**: use `cell_editable_bg` for empty editable cells, `cell_editable_filled_bg` for filled+editable.
- **Keyboard auto-open**: all score input dialogs must call `dialog.window?.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_VISIBLE)` and `editText.requestFocus()` so the soft keyboard opens immediately on dialog show.

---

## Localisation

- The app supports **English** (`res/values/strings.xml`) and **French** (`res/values-fr/strings.xml`).
- Every new string must be added to **both files**.
- Never hardcode text in Kotlin code or XML layouts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trivialloop/scorehub](https://github.com/trivialloop/scorehub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
