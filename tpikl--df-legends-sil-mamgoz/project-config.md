---
trigger: always_on
description: This project extracts and transforms Dwarf Fortress legends data into readable narratives and story documents.
---

# CLAUDE.md

This project extracts and transforms Dwarf Fortress legends data into readable narratives and story documents.

## Project Structure

```
df-claude-legends/
├── world/                        # Dwarf Fortress world data
│   ├── SilMamgoz/                # Raw DF save files (do not touch)
│   ├── legends/                  # Legends exports organized by year
│   │   ├── 500/                  # Year 500 exports
│   │   │   ├── *-legends.xml     # Standard legends export from DF
│   │   │   └── *-legends_plus.xml # Extended export from DFHack
│   │   └── 501/                  # Year 501 exports (latest)
│   │       ├── *-legends.xml
│   │       └── *-legends_plus.xml
│   └── parsed/                   # Split XML files (see below)
├── stories/                      # Generated narratives and character studies
│   └── {character}/              # Per-character story folders
├── scripts/                      # Utility scripts
│   ├── parse_legends.py          # XML splitter script
│   ├── lookup_figure.py          # Find figures by ID, name, or race
│   ├── lookup_site.py            # Find sites by ID, name, or type
│   ├── lookup_entity.py          # Find entities/civilizations
│   ├── lookup_events.py          # Find events by figure, site, type, year
│   ├── lookup_creature.py        # Translate creature IDs to names
│   └── figure_history.py         # Generate full timeline for a figure
├── .claude/                      # Claude Code configuration
│   └── settings.json             # Permissions for auto-approved scripts
└── README.md
```

## Folder Guidelines

### `world/SilMamgoz/`
Raw Dwarf Fortress save data. **Leave this alone.** Contains compressed game data.

### `world/legends/{year}/`
Legends exports organized by in-game year. Each folder contains:
- `*-legends.xml` - Standard legends export from DF
- `*-legends_plus.xml` - Extended export from DFHack with additional data

The latest year folder contains the most complete history. Keep these as source of truth but **prefer using parsed/ files** for lookups.

### `world/parsed/` (Preferred for lookups)
Pre-split XML files organized by category. **Use these instead of the monolithic XMLs** to avoid loading unnecessary data.

```
parsed/
├── world_info.xml              # World name: "Sil Måmgoz" (The Plane of Dragons)
├── geography/                  # Regions, rivers, mountains, underground
├── entities/                   # Civilizations, sites, populations
├── figures/                    # Historical figures
│   ├── historical_figures_all.xml      # Complete (2.2 MB)
│   ├── historical_figures_plus.xml     # Extended data from DFHack
│   └── by_race/                        # Split by race for targeted lookups
│       ├── dwarf.xml (434 figures, 1 MB)
│       ├── human.xml (264 figures, 620 KB)
│       ├── kobold.xml (28 figures)
│       ├── elf.xml (19 figures)
│       └── ... (100+ race files)
├── events/                     # Historical events (17,224 total)
│   ├── by_type/                        # Split by event type
│   │   ├── hf_died.xml (1,006 deaths)
│   │   ├── hf_simple_battle_event.xml (5,573 battles)
│   │   ├── creature_devoured.xml (4,810 events)
│   │   └── ... (60+ event types)
│   ├── by_year/                        # Split by 50-year ranges
│   │   ├── years_0000-0049.xml
│   │   ├── years_0050-0099.xml
│   │   └── ...
│   ├── event_collections.xml           # War/battle groupings
│   └── event_relationships.xml         # Figure relationship events
├── culture/                    # Artifacts, writings, music, poetry, dance
└── reference/                  # Creature definitions, historical eras
```

See `world/parsed/INDEX.md` for complete file listing with sizes.

### `stories/`
Contains narrative documents crafted from the legends data. Organized by subject (character, civilization, event, etc.).

## Working with This Project

### Lookup Scripts (Preferred Method)
Use these scripts for fast, formatted lookups. They read from `world/parsed/` and resolve IDs to names automatically.

```bash
# Find a figure by ID or name
python3 scripts/lookup_figure.py 123
python3 scripts/lookup_figure.py "sodel"
python3 scripts/lookup_figure.py --race dwarf --brief

# Find a site by ID or name
python3 scripts/lookup_site.py 8
python3 scripts/lookup_site.py "longpaints"
python3 scripts/lookup_site.py --type fortress

# Find entities/civilizations (includes population counts)
python3 scripts/lookup_entity.py 26
python3 scripts/lookup_entity.py --list
# Output: ENTITY #30: The Unswerving Fells
#         Race: Elf
#         Type: Civilization
#         Population: 20 elf

# Find events (combinable filters)
python3 scripts/lookup_events.py --figure 123
python3 scripts/lookup_events.py --site 8 --year 50-100
python3 scripts/lookup_events.py --type hf_died --limit 50
python3 scripts/lookup_events.py --list-types

# Translate creature IDs to names
python3 scripts/lookup_creature.py NIGHT_CREATURE_3
python3 scripts/lookup_creature.py --list-special

# Get complete timeline for a figure
python3 scripts/figure_history.py 107
python3 scripts/figure_history.py "fimshel"
python3 scripts/figure_history.py 45 --brief
```

### Manual Lookups (parsed/ files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tpikl/df-legends-sil-mamgoz](https://github.com/Tpikl/df-legends-sil-mamgoz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
