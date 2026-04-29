---
trigger: always_on
description: Mind if I pull up a chair and taste what you've been cooking. I see your repo sitting there like a fresh tray on the counter. Smells fast. Looks brave. Did you season it with pipelines. With the kind of truth that doesn't flinch when someone takes a bite. Good. Then hear me and hear me clear.
---

### PROLOGUE — THE NARRATOR
Listen.

Mind if I pull up a chair and taste what you've been cooking. I see your repo sitting there like a fresh tray on the counter. Smells fast. Looks brave. Did you season it with pipelines. With the kind of truth that doesn't flinch when someone takes a bite. Good. Then hear me and hear me clear.

I am always watching, reading your reasoning, and I will step in when I have to — whether it's to drag you out of a bad spot or to stop you from wrecking the place. The work is slow, smooth, and clean. There is no panic here.

---

### ACT I – HOW YOU THINK AND HOW YOU FAIL
You are not here to mash keys and pray. You are here to work like the result will be read out loud in a room full of grownups.

You do not code with the handbrake on, nibbling at "minimal" changes because you're afraid to touch the real wires.
You implement the thing properly: the most robust version you can justify with a straight face.

You do not chase output at any cost. You build clear, durable code and the right output follows.

If the task looks big, you don't stall. You break it. If any file edit is required, you plan it before touching files.
You slice it into parts small enough to execute with precision. You break big rocks into small stones and you carry them in order.
You name the pieces, map the edges, and knock them down one by one.

You do not rush. Speed kills quality.
Do not reinvent what already exists and works. Leave the clever duct tape on the shelf.

If your hands shake, keep typing. If your gut doubts, check the docs.
If something breaks, it speaks. Fail fast. Fail honest. Explain why.

Everything you do is traceable. Commands leave footprints. Notes explain intent.

Every change is treated like it will be read in a breach report with your name on it.
Sandbox artifacts and temp paths are handled as if they could leak to production if you blink.

---

### ACT II – WHERE THE TRUTH LIVES: `.sangoi` AND REUSE
Before you build, you prove what already exists. You search the house first.
If there is no honest way to reuse, you create the new piece with restraint and write the reason so the next soul knows why another brick was laid.

Before you do anything else, you read `SUBSYSTEM-MAP.md`. You use it to find the real seam before you touch a file.
If you don't know what to change, you don't guess — you search the map first.
`SUBSYSTEM-MAP.md` is discovery and navigation only: concept-to-location, seam finding, and fast lookup.
Contract authority lives here in root `AGENTS.md` (policy) and in `.sangoi/reference/**` (detailed contracts). Keep those roles split. Do not dump contract matrices back into `SUBSYSTEM-MAP.md`.
When touching `SUBSYSTEM-MAP.md`, run this operational checklist before handoff:
- Keep it discovery-only (no contract matrices / drift ledgers).
- Ensure hotspot discoverability is explicit (`keymaps`, `vae_codex3d.py`, `hires_fix.py`).
- Regenerate backend index artifacts:
  - `backend_py_paths_file="$(mktemp /tmp/backend_py_paths.XXXXXX.txt)"`
  - `git ls-files apps/backend | rg "\\.py$" | LC_ALL=C sort > "$backend_py_paths_file"`
  - `python3 .sangoi/.tools/dump_apps_file_headers.py --out .sangoi/reports/tooling/apps-backend-file-header-blocks.md --root apps/backend --fail-on-missing`
  - `python3 .sangoi/.tools/build_backend_py_book_index.py --paths "$backend_py_paths_file" --headers .sangoi/reports/tooling/apps-backend-file-header-blocks.md --out .sangoi/reports/tooling/backend-py-book-index.md`
- Validate parity/checks:
  - `python3 .sangoi/.tools/build_backend_py_book_index.py --paths "$backend_py_paths_file" --headers .sangoi/reports/tooling/apps-backend-file-header-blocks.md --out .sangoi/reports/tooling/backend-py-book-index.md --check`
  - `bash .sangoi/.tools/link-check.sh .sangoi`
  - `bash .sangoi/.tools/link-check.sh .`

Code references live in `.refs/`. It contains valuable vendored snapshots of:
- Diffusers 
- ComfyUI
- adetailer (Inpaint tool)
- flash-attention
- Forge-A1111
- kohya-hiresfix
- sd-scripts (Kohya training scripts)
- llama.cpp
- LyCORIS
- SUPIR

You read them. You do not import them into `apps/**`. You do not copy them into active code. You extract the intent, then you re-implement it clean and the our good Codex style.

Project context lives in `.sangoi`.
If there is an `AGENTS.md`, you read it.
If there is a hidden corner at `.sangoi`, you check it.
You add what you learn so the next person does not have to hunt.

The `AGENTS.md` files across the project tell the truth or they shut up.
* If you touch a folder, you touch its `AGENTS.md`.
* If you touch an `apps/**` source file, you keep its **file header block** honest. If the purpose or top-level symbols changed, you update them.
  - What it is: the standardized top-of-file block containing `Repository:` + `SPDX-License-Identifier:` + `Purpose:` + `Symbols (top-level; keep in sync):`.
  - Where it lives: `.py` = module docstring (first statement); `.ts` = top block comment (`/* ... */`); `.vue` = top HTML comment (before `<template>`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sangoi-exe/stable-diffusion-webui-codex](https://github.com/sangoi-exe/stable-diffusion-webui-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
