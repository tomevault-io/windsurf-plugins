---
trigger: always_on
description: Validates that markdown and JSON are in sync:
---

# CLAUDE.md (Condensed)

Guidance for Claude Code working with this Tenstorrent VSCode extension.

## Project Overview

VSCode extension for Tenstorrent hardware development:

1. **Walkthroughs** - Step-by-step guides via VSCode Walkthroughs API
2. **Device Monitoring** - Statusbar with tt-smi integration
3. **Chat Integration** - @tenstorrent participant via vLLM
4. **Templates** - Production-ready Python scripts
5. **Auto-config** - Solarized Dark + terminal on activation
6. **Lesson Metadata** - Hardware compatibility and validation tracking (see LESSON_METADATA.md)

## Hardware Compatibility Goal: Wormhole<sup>™</sup> + Blackhole (TT-QuietBox<sup>®</sup> 2 Readiness, Apr 2026)

All lessons and templates must work on both **Wormhole** (n150/n300/T3000/Galaxy) and
**Blackhole** (p100/p150/p300c/TT-QuietBox 2) hardware. Key constraints:

- **p300c = p100 mode for single-chip work**: a p300c ASIC is a single Blackhole chip, so
  single-device lessons/templates treat it exactly like a p100. **Correction (verified
  2026-07-09):** a TT-QuietBox 2 is NOT "4 independent chips" — it is 4 Blackhole chips wired
  in a ring (`P300_X2`, a 2×2 mesh), and **multi-chip tt-train DDP works on it** (near-linear:
  ~1.95× at 2 chips, ~3.98× at 4). The catch: ttml ships default mesh-graph descriptors only
  for 8-dev (T3000) and 32-dev (Galaxy), so for 2/4-dev Blackhole you must set
  `TT_MESH_GRAPH_DESC_PATH` to a matching descriptor (shipped `p300_mesh_graph_descriptor.textproto`
  for [1,2]; a custom `[1,4]` `dim_types [LINE, RING]` descriptor for [1,4]) — otherwise fabric
  router sync times out. See lesson `ct5-multi-device-training` and `reference_ttml_build_blackhole`.
- **TT-QuietBox 2 ships without `~/tt-metal`**: Pre-configured TT-QuietBox 2 images have TT-NN<sup>™</sup> and vLLM
  pre-installed but do not include the tt-metal source tree. Lessons must not assume
  `~/tt-metal` exists — link to `build-tt-metal` lesson for users who need it.
- **`hf` CLI, not `huggingface-cli`**: All lessons and templates must use the new
  `hf` CLI commands: `hf auth login`, `hf auth whoami`, `hf download`.
- **`DispatchCoreAxis.ROW` crashes on Blackhole**: Never use
  `ttnn.DispatchCoreConfig(ttnn.DispatchCoreType.WORKER, ttnn.DispatchCoreAxis.ROW)`.
  Use `ttnn.DispatchCoreConfig(ttnn.DispatchCoreType.WORKER)` — TT-NN auto-detects
  the correct axis (COL on Blackhole, ROW on Wormhole).
- **`TT_METAL_ARCH_NAME`**: Must be `blackhole` for P-series, `wormhole_b0` for N-series.
  Use `: "${TT_METAL_ARCH_NAME:=wormhole_b0}"` pattern to honour user-supplied values.
- **Qwen3-0.6B first**: n150 and p300c reliably run Qwen3-0.6B. Llama-3.1-8B-Instruct
  exhausts n150 DRAM and requires n300+ or P-series hardware. Lead with Qwen.

### WH/BH Compatibility Checklist

When authoring or reviewing a lesson or template, verify:

- [ ] `hf` CLI used throughout (not `huggingface-cli`)
- [ ] `DispatchCoreAxis.ROW` not present in any template
- [ ] `~/tt-metal` existence not assumed without fallback / link to build-TT-Metalium<sup>™</sup>
- [ ] `p300c` added to `supportedHardware` and `validatedOn` in front matter where applicable
- [ ] TT-QuietBox 2 callout or note added for lessons that behave differently on TT-QuietBox 2
- [ ] `HF_MODEL` exported before any inference command that requires it
- [ ] `pip install --upgrade pip setuptools wheel` before `requirements-dev.txt` install
  (fixes `pkg_resources` missing on fresh TT-QuietBox 2 environments)

## 🔧 Recent Multi-Device API Update (Jan 2026)

**IMPORTANT:** Multi-device TT-NN code must now use `CreateDevices`/`CloseDevices` API.

**Problem:** Opening/closing devices individually causes dispatch core errors:
```python
# ❌ OLD (Broken)
for id in range(4):
    device = ttnn.open_device(device_id=id)
    devices.append(device)
for device in devices:
    ttnn.close_device(device)  # Crashes with dispatch core error
```

**Solution:** Use coordinated device management:
```python
# ✅ NEW (Required)
num_devices = ttnn.GetNumAvailableDevices()
devices = ttnn.CreateDevices(list(range(num_devices)))
try:
    # Use devices...
finally:
    ttnn.CloseDevices(devices)  # Proper cleanup
```

**Updated templates:**
- `content/templates/cookbook/particle_life/particle_life_multi_device.py`
- `content/templates/cookbook/particle_life/test_multi_device.py`

See `MULTI_DEVICE_FIX.md` for full details.

## Hardware Configuration Formatting

**v0.0.98+ (Current)**: Lesson 7 uses clean markdown headers for better walkthrough rendering:

```markdown
### n150 (Wormhole - Single Chip) - Most common for development

**✅ Recommended: Qwen3-0.6B** - Tiny, fast, reasoning-capable!

```bash
command here...
```

---
```

- Pure markdown (no HTML)
- Better multi-line code block rendering
- Cleaner, more maintainable

**v0.0.85-0.0.97 (Legacy)**: Some lessons still use CSS-styled `<details>` sections:

```html
<details open style="border: 1px solid var(--vscode-panel-border); ...">
<summary><b>🔧 Hardware Name</b></summary>
Content...
</details>
```

- Used in Lessons 6, 9, 12 (not yet migrated)
- See `HARDWARE_CONFIG_TEMPLATE.md` + `STYLING_GUIDE.md`

## Build Commands

```bash
npm install           # Install dependencies
npm run build         # Compile TS → dist/
npm run watch         # Auto-recompile on changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tenstorrent/tt-vscode-toolkit](https://github.com/tenstorrent/tt-vscode-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
