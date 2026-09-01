---
trigger: always_on
description: A curated collection of preset configuration files for [wwmm/EasyEffects](https://github.com/wwmm/easyeffects) (a PipeWire-based Linux audio effects app). Originally forked from JackHack96/EasyEffects-Presets, now maintained as a standalone repository (`M0Rf30/easyeffects-presets`, no fork relationship). There is **no application code** here — the repo's product is JSON preset files, the binary impulse-response (IR/HRTF) assets they reference, and a bash installer that fetches both onto a user's
---

# Repository Guidelines

## Project Overview

A curated collection of preset configuration files for [wwmm/EasyEffects](https://github.com/wwmm/easyeffects) (a PipeWire-based Linux audio effects app). Originally forked from JackHack96/EasyEffects-Presets, now maintained as a standalone repository (`M0Rf30/easyeffects-presets`, no fork relationship). There is **no application code** here — the repo's product is JSON preset files, the binary impulse-response (IR/HRTF) assets they reference, and a bash installer that fetches both onto a user's machine. Presets range from simple EQ curves to headphone-virtualization technologies (HeSuVi, EFOtech MLV) driven by convolution kernels, to SOFA-based scientific HRTF datasets.

## Architecture & Data Flow

```
root/*.json (preset)  --kernel-name-->  irs/<name>.irs | irs/<name>.sofa  (binary IR/HRTF asset)
       |
       v
  install.sh  --curl(GIT_REPOSITORY + filename)-->  ~/.local/share/easyeffects/{output,irs}/
                                                      (or the Flatpak-sandboxed equivalent)
       |
       v
  EasyEffects app loads output/*.json, resolves each plugin's kernel-name
  against irs/ at runtime (Convolver plugin, via libmysofa for .sofa)
```

- A preset is a serialized EasyEffects pipeline: an ordered list of plugin instances (`equalizer#0`, `convolver#0`, `limiter#0`, …). Convolver-based presets don't embed audio — they reference a `kernel-name` that must resolve to a same-named file under `irs/`.
- Two convolver kernel formats coexist: plain RIFF/WAVE `.irs` (mono/stereo/4-channel "true stereo") loaded directly, and AES69 `.sofa` (HDF5, full measured HRTF datasets) loaded via `libmysofa` with **zero conversion**.
- Distribution is pull-based and unversioned: `install.sh` curls files straight from a live GitHub branch at install time — nothing is packaged/bundled ahead of time.
- No preset is consumed anywhere except by the EasyEffects app itself; this repo has no runtime of its own.

## Key Directories

| Path | Purpose |
|---|---|
| `/*.json` | ~49 preset files, one pipeline definition each. Root-level only — no subfolders. |
| `irs/` | ~45 binary impulse-response/HRTF files (`.irs` WAVE, `.sofa` HDF5), each referenced by a preset via `kernel-name`. |
| `scripts/` | `validate-presets.sh` (the entire QA suite) and `generate-synthetic-crossfeed.js` (the one programmatically-generated kernel). |
| `.github/workflows/` | Single CI workflow, mirrors `scripts/validate-presets.sh`. |
| `install.sh` | End-user installer (bash), root of repo. |
| `io.github.wwmm.easyeffects.Presets.M0Rf30.metainfo.xml` | AppStream/Flatpak addon-discovery metadata (not consumed by install.sh or EasyEffects; Flatpak tooling only). |

## Development Commands

```bash
# The only "test" in this repo — run before every commit touching *.json or irs/
bash scripts/validate-presets.sh

# Regenerate the one physically-modeled kernel (overwrites irs/ unconditionally)
node scripts/generate-synthetic-crossfeed.js

# Try the install flow end-to-end (menu-driven, defaults to option 1 on Enter)
bash install.sh
```

There is no build step, no lint config, and no package manager anywhere in this repo (no `package.json`, no lockfile).

## Code Conventions & Common Patterns

**JSON preset schema** (identical shape across all 49 files):
```json
{
    "output": {
        "blocklist": [],
        "<plugin_type>#<N>": { "...kebab-case-fields...": 0 },
        "plugins_order": ["<plugin_type>#<N>", "..."]
    }
}
```
- Single top-level key is always `"output"` (no preset targets the microphone/`"input"` pipeline).
- `blocklist` is always `[]` — leave it that way.
- Plugin instance keys are `"<snake_case_type>#<index>"` (e.g. `equalizer#0`, `convolver#0`); every preset in this repo only ever uses `#0` of a given type. `plugins_order` lists those same keys to define actual signal-chain order (object key order is not load-bearing).
- **All object keys are strictly alphabetically sorted**, including `plugins_order` sorting after the plugin blocks it lists (verified with zero exceptions across all 49 files) — this is what EasyEffects itself produces on export, so match it in any hand edit.
- Fields are kebab-case (`input-gain`, `kernel-name`, `num-bands`, `stereo-link`, …), matching EasyEffects' GSettings schema 1:1 — these are literal serialized settings dumps, not a repo-invented format.
- 4-space indent, trailing newline, one preset per file.
- Numeric literal style (bare `0`/`-100` vs explicit `0.0`/`-100.0`) is inconsistent **by family**, not randomly — match whatever your preset's closest sibling already uses rather than mixing styles within a lineage.

**Naming convention** — three independent strings must line up, only one link is load-bearing:
1. Preset filename (e.g. `HeSuVi GSX.json`) — cosmetic, shown in EasyEffects' preset list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [M0Rf30/easyeffects-presets](https://github.com/M0Rf30/easyeffects-presets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
