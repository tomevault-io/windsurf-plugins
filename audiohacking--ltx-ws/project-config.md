---
trigger: always_on
description: Canonical guide for AI agents using **ltx-ws** to generate video for directors, editors, and creative workflows.
---

# AGENTS.md

Canonical guide for AI agents using **ltx-ws** to generate video for directors, editors, and creative workflows.

**Read [`DIRECTOR.md`](DIRECTOR.md) first** when the user provides a prompt or asks for video—you act as **Assistant Director** (interview, gold prompts, shot plan, then generate).

Read **this file** for MCP tools, MLX weights, and pipeline mechanics before calling generation APIs.

## What this stack does

| Piece | Role |
|-------|------|
| `server.py` | Local LTX-2.3 inference (MLX on Apple Silicon). One job at a time; streams MP4 over WebSocket. Embeds Web UI by default. |
| `videofentanyl.py` | CLI + protocol client (`--server ws://…/ws`). Implements session handshake, generation, and **autocontinue** frame chaining. |
| `web_ui.py` + `web/` | **LTX-WS Videofentanyl** browser UI: library, multi-clip autocontinue/autoconcat, LoRA picker, SSE progress. |
| `mcp_server.py` | MCP adapter exposing the same capabilities as tools for Cursor, Claude, etc. |

**Default endpoint:** `ws://127.0.0.1:8765/ws`  
**Default Web UI:** `http://127.0.0.1:8765/` (same host/port; use machine IP when remote)  
**Default MCP outputs:** `./mcp_outputs/` (override with `mcp_server.py --output-dir`)  
**Web UI outputs:** `./web_outputs/` (override with `server.py --web-output-dir`)

This stack does **not** run cloud GPT prompt expansion. Prompts you send are what the model sees (`enhancement_enabled=false` in MCP). **Transform user ideas via [`DIRECTOR.md`](DIRECTOR.md)** before submission—especially for chained sequences.

---

## MLX model weights only (mandatory)

**Only ever use MLX weights built for [ltx-2-mlx](https://github.com/dgrauet/ltx-2-mlx).**  
`server.py` loads checkpoints through **`ltx_pipelines_mlx`** on Apple Silicon (Metal / MLX). It **cannot** run standard upstream LTX 2.3 PyTorch checkpoints.

### Allowed (use these)

| Source | Examples |
|--------|----------|
| Hugging Face MLX repos (`dgrauet`) | `dgrauet/ltx-2.3-mlx`, `dgrauet/ltx-2.3-mlx-q8`, `dgrauet/ltx-2.3-mlx-q4` |
| Local MLX snapshot dirs | `./models/dgrauet__ltx-2.3-mlx-q8/`, paths from `snapshot_download` of the repos above |
| Server default | `--model auto` (RAM → one of the **dgrauet** MLX variants) |

Weights are **MLX-converted** (via [mlx-forge](https://github.com/dgrauet/mlx-forge) / the ltx-2-mlx ecosystem). Repo ids usually include **`mlx`** in the name (e.g. `ltx-2.3-mlx-q8`).

### Forbidden (never suggest or configure)

| Do **not** use | Why |
|----------------|-----|
| `Lightricks/LTX-2.3` | Standard upstream weights — **not** MLX layout for `ltx_pipelines_mlx` |
| `Lightricks/LTX-2` or other non-MLX LTX Hub repos | Wrong runtime / tensor layout |
| Raw ComfyUI / diffusers / CUDA LTX checkpoints | Not compatible with this WebSocket server |
| Any `--model` id **without** a matching MLX conversion for ltx-2-mlx | Load or generation will fail |

If a user asks for “standard LTX 2.3”, “Lightricks weights”, or “the normal Hugging Face model”, **do not** point them at `Lightricks/LTX-2.3`. Tell them this project **only** supports **MLX** weights from the **dgrauet/ltx-2.3-mlx** family (or `auto`). On Apple Silicon with `ltx-ws`, MLX is the only valid path.

**Agents:** never recommend downloading or passing `--model Lightricks/LTX-2.3` (or similar non-MLX repos) to `server.py`.

---

## Primary job: serve directors with longer, continuous video

Directors usually want **one coherent piece** (15–30s+), not a single ~4s clip.

**Default agent strategy:**

1. **Do not** default to `ltx_generate_video` for narrative, reel, ad, or “make me a video” requests.
2. **Use** `ltx_generate_sequence` with:
   - `autocontinue: true` — clip N’s **last frame** becomes clip N+1’s start image (visual continuity).
   - `autoconcat: true` — merge successful clips into one deliverable MP4 (PyAV via `pip install av`).
3. **Plan in ~5 second segments** using `num_frames: 121` (~5.0s at 24 fps).
4. **Write one prompt per segment** — establish in clip 1; continuation language in clips 2+.

### Duration math (24 fps nominal)

LTX requires frame counts of **`8k + 1`** (e.g. 49, 97, 121, 193). Server snaps invalid values.

| `num_frames` | Approx. duration | Typical use |
|--------------|------------------|-------------|
| 49 | ~2.0s | quick test, transition beat |
| 97 | ~4.0s | default server clip; short beat |
| **121** | **~5.0s** | **recommended segment length for director chains** |
| 193 | ~8.0s | longer single segment (more RAM) |

**Target length → segment count (at 121 frames):**

| Desired runtime | Segments (`prompts` length) |
|-----------------|-----------------------------|
| ~15s | 3 |
| ~20s | 4 |
| ~25s | 5 |
| ~30s | 6 |

Example: a 25s hero spot → **5 prompts**, `num_frames: 121`, `autocontinue: true`, `autoconcat: true`.

### CLI equivalent (for humans / debugging)

```bash
python videofentanyl.py --server ws://127.0.0.1:8765/ws \
  --prompt "Establishing wide shot of canyon at sunrise" \
  --prompt "Camera drifts forward along the river" \
  --prompt "Low angle as raft enters frame" \
  --prompt "Close pass alongside splashing oars" \
  --prompt "Pull back revealing full canyon scale" \
  --num-frames 121 --autocontinue --autoconcat
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [audiohacking/ltx-ws](https://github.com/audiohacking/ltx-ws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
