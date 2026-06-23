---
trigger: always_on
description: AI agent instructions for the `signal_light_demo` TuyaOpen project.
---

# AGENTS.md — signal_light_demo

AI agent instructions for the `signal_light_demo` TuyaOpen project.

## Project snapshot

- **Name**: `signal_light_demo`
- **Platform**: `t5`
- **Framework**: `base`
- **Source**: Embedded firmware scaffolded from `TuyaOpenSDK/tools/app_template/base` into `source/embedded/`. Build with `cd source/embedded && tos.py build`.

## Project layout

```
signal_light_demo/
├── .tuyaopen/
│   ├── project.json        # name, version, type, framework; ai.intent only when AI-created
│   ├── status.json         # lifecycle: scaffolded → configured → built → flashed
│   ├── architecture.json   # surfaces, modules, components, dependencies
│   ├── dependencies.lock.json   # ecosystem libs (when installed via IDE)
│   ├── ide/                # IDE-managed runtime files (do not edit)
│   │   ├── bin/tuya-devplat-cli   # wrapper script — use full path outside VSCode
│   │   ├── platform.json          # hardware capability spec for the active platform (chip-level)
│   │   └── board.json             # board-level hardware spec (onboard components, pre-allocated pins)
│   └── platform/           # Tuya Platform product snapshots (IDE)
│       └── product-<pid>.json
├── tuyaopen.project.ini    # human-readable project config (IDE-maintained)
├── source/
│   ├── embedded/           # firmware — build with tos.py
│   │   ├── include/        # project header files
│   │   └── src/
│   │       └── tuya_app_main.c
│   └── miniapp/            # panel/mini-app placeholder
├── .vscode/
│   └── settings.json       # CMake source + build directory
├── .agents/skills/         # AI agent skill catalogue (Agent Skills standard)
├── .gitignore
├── AGENTS.md
├── CLAUDE.md
└── README.md
```

## `.tuyaopen/` descriptor contract

**`project.json`**
- `name`, `version`, `type`, `framework` — project identity
- `ai.intent` — user's original intent _(only present for AI-assisted scaffold — absent in normal projects)_
- `ai.assistedScaffold` — true when created via "Create with AI" _(optional — absent in normal projects)_
- `ai.skills` — reserved for future use; **do not rely on it to discover installed skills**. Scan `.agents/skills/` directly — each subdirectory (flat or one level deep) whose `SKILL.md` exists is an installed skill.

**`status.json`** — advance `lifecycle` as the project progresses:
`"scaffolded"` → `"configured"` → `"built"` → `"flashed"`

**`architecture.json`** — update when structure changes:
- `surfaces.embedded` — platform, chip, board, entrypoint, RTOS config, hardware peripherals
- `surfaces.miniapp` — panel kind and entrypoint (null until Panel SDK runs here)
- `modules` — logical modules with source paths and surface
- `dependencies` — external libraries (TuyaOpen core, etc.)

**`dependencies.lock.json`** — created when the IDE installs a library from Library → Ecosystem. Canonical pinned versions; commit to git. The `[ecosystem]` section of `tuyaopen.project.ini` mirrors this file.

**`.tuyaopen/platform/`** — Tuya Platform product snapshots (IDE-owned, read-only). One file per active PID.

- **Binding** — active PID is `tuyaopen.project.ini` → `[product] pid`; the filename is informational.
- **Envelope** — `schemaVersion` (1 or 2), `pid`, `fetchedAt`, `source`, `detail`, `dpSchema`, `fetchError`.
- **Unwrap** — `detail` and `dpSchema` may be wrapped `{ ok, data }`; always read as `field.data ?? field`.
- **Active DPs** — read `dpSchema.dps[]` where `selected === true`; skip all others.
- **Cloud functions** — `dpSchema.uiConfig.bic[]` lists BIC cloud functions enabled for this product.
- **Never invent** — if missing or `fetchError` is set, ask the developer to sync in the IDE.
- **Mutations** — use `tuya-devplat-cli` commands or skill `tuyaopen-dp-add`; never edit the JSON directly.

Deep reference: `.agents/skills/tuya-iot-platform/references/platform-product-snapshot.md`
_(Install skill **Tuya IoT Platform** from the IDE Skills panel if the path does not exist.)_

## `tuyaopen.project.ini`

Human-readable project descriptor at the project root. Maintained by the TuyaOpen IDE; hand-editing is allowed and unknown keys are preserved on round-trips.

| Section | Purpose |
|---|---|
| `[project]` | `name`, `version`, `type`, `schema` — project identity |
| `[platform]` | `target` (`t5` \| `esp32` \| `linux` \| `raspberrypi`), `sdk`, `sdk_version` |
| `[build]` | `output`, `toolchain` |
| `[board]` | Present when a board was selected at creation: `id`, `kconfig_id`, `config_file` (optional fields) |
| `[features]` | Optional feature toggles (`key = value`) |
| `[product]` | Present when bound to a Tuya Platform product: `pid = …` |
| `[ecosystem]` | Present when ecosystem libraries are installed: `owner/name = ^x.y.z` (mirrors `dependencies.lock.json`) |

Prefer updating structured JSON under `.tuyaopen/` for AI-facing contract fields (`project.json`, `architecture.json`). Use the INI when a skill or workflow explicitly targets project config, or when mirroring values the IDE already wrote (product PID, ecosystem constraints).

## Build & flash (embedded)

```bash
cd source/embedded
tos.py check     # verify SDK environment
tos.py build     # compile firmware

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robeortZ/signal_light_demo](https://github.com/robeortZ/signal_light_demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
