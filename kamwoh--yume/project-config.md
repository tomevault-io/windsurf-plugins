---
trigger: always_on
description: You are working on **Yume**: a JSON-declarative content layer over Godot
---

# Yume (夢) — JSON-driven game framework on Godot

You are working on **Yume**: a JSON-declarative content layer over Godot
4.6.1 that lets non-programmers (and LLMs) generate working games without
writing GDScript per-game. Per ADR 0021, the engine ships a fixed verb
set (the "seven primitives" + a small interpreter); each game's mechanics
are pure JSON.

## Quick Start

This repo ships the **framework** (engine, skills, scaffolding scenes,
shared engine libraries) but NOT specific game demos. Demos live at
`godot/data/demo_<name>/` and are gitignored — generate locally via the
pipeline below or copy from another working tree.

To generate a new game from a prose pitch:

```
/yume-design "a roguelike where vampires steal HP from light sources" --autonomous
```

The `/yume-design` skill orchestrates text → GDD → world plan → level
design → rules → JSON → assets → QA. Output lands at
`godot/data/demo_<slug>/`.

To run a locally-generated demo:

```bash
./scripts/play.sh <name>            # falls back to scenes/play.tscn --game=<name>
./scripts/play.sh <name> --capture  # auto-capture for visual QA
```

## Framework structure

```
yume/
├── .claude/                            ← Skills + rules + settings
│   ├── skills/yume-*/SKILL.md          (38 specialist skills, Tier 2.6)
│   └── rules/                          (path-scoped invariants)
├── godot/    ← Godot project (engine + scaffolding; demos gitignored)
│   ├── data/                           ← shared engine libraries (TRACKED)
│   │   ├── shapes.json                 (code-draw shape library)
│   │   ├── meshes.json                 (3D mesh library)
│   │   └── sounds.json                 (procedural SFX library)
│   ├── data/demo_<name>/               (per-game content — NOT TRACKED, gitignored)
│   │   ├── entities/                   (definitions + initial instances)
│   │   ├── world/rules.json          (world physics rules — ADR 0009)
│   │   ├── game/goals.json             (game logic — win/score/transition)
│   │   ├── game/flow.json              (multi-level progression — ADR 0006)
│   │   ├── levels/<n>/                 (per-level entities + rules)
│   │   ├── world/state.json            (initial _engine entity state — ADR 0047)
│   │   ├── world/zones.json            (zone-state primitive — ADR 0031, optional)
│   │   ├── audio/cues.json             (semantic-event → SFX mapping)
│   │   ├── ui/strings.json             (localizable text)
│   │   ├── scene.json                  (camera + lighting + ground + tick_seconds)
│   │   ├── screens.json                (title/pause/etc. — ADR 0011, optional)
│   │   ├── save_policy.json            (what persists — ADR 0010, optional)
│   │   ├── settings_schema.json        (settings — ADR 0013, optional)
│   │   └── tutorial.json               (overlay sequencing — ADR 0012, optional)
│   ├── scripts/engine/                 (engine: rule, query, effect, scheduler, etc.)
│   ├── scenes/                         (TRACKED scaffolding only)
│   │   ├── play.tscn                   (universal launcher — `--game=<name>`)
│   │   ├── test_main.tscn              (engine unit tests)
│   │   └── scenario_test.tscn          (per-game scenario test runner)
│   └── project.godot
├── docs/                               ← Documentation (active)
│   ├── 30_framework_primitives.md      (the contract — invariant-bearing)
│   ├── 31_text_to_game_pipeline.md     (Tier 2.5 strategic plan)
│   ├── 32_mda_for_yume.md              (Mechanics → Dynamics → Aesthetics)
│   ├── adr/NNNN-*.md                   (architecture decisions)
│   ├── engine-reference/               (api manifest, Godot pinning)
│   ├── games/<name>/                   (per-game GDDs, plans, reviews)
│   └── timeline/                       (decision diary)
├── scripts/play.sh                     ← Run a demo (sync + launch)
├── tools/gen_api_manifest.py           ← Regenerate engine API manifest
├── task_plan.md                        ← Index → .claude/plan/{backlog,archive}.md
└── CLAUDE.md (this file)
```

## How the engine works

Yume is **primitives + interpreter** (Invariant #8). The engine ships a
fixed vocabulary in GDScript; all game-specific behavior lives in JSON.

**Seven primitives** (ADR 0001):

1. **Entity** — JSON dict with id, tags, properties, state, position
2. **Tag** — string membership label (no class hierarchy)
3. **Rule** — `{trigger, query, effect}` triple
4. **Trigger** — when (tick / contact / signal / input / spawn / despawn / relation_changed)
5. **Effect** — what (state_set / spawn / remove / transform / relate / velocity_set / emit / ... — full list in `docs/engine-reference/api-manifest.json`)
6. **Query** — entities matching tags + state + radius + relations
7. **Relation** — typed directed edge between entities

**ADR 0021** (foundational): Yume = JSON layer over Godot. Engine never
reimplements what Godot already does well — it EXPOSES Godot's
capabilities through JSON-declarative primitives. Each new capability
is a "capability-exposure ADR" (e.g. ADR 0011 for Control nodes, ADR
0010 for FileAccess+JSON, future ADR 0022 for PhysicsServer3D).

## Tick rate is the engine's heartbeat (2026-05-16)

**The tick is the engine's clock — never change it as a balance knob.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kamwoh/yume](https://github.com/kamwoh/yume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
