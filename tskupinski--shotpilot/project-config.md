---
trigger: always_on
description: Agent-first pipeline from drone footage to an edited film: smoothness analysis
---

# Shotpilot

Agent-first pipeline from drone footage to an edited film: smoothness analysis
(OpenCV), aesthetic and pacing review, cutting selects, content tags, sequencing,
splicing (ffmpeg), music (AI generation + mux) and YouTube publishing assets
(`shot publish`: thumbnail + title + description). Current stage: the full cycle
selection → pacing → montage with crossfade transitions → music → publishing.

This file is the canonical instruction set for every coding agent (Claude Code
reads it via `CLAUDE.md`, Codex CLI and OpenCode read it directly). Private
machine- and footage-specific facts live in `CLAUDE.local.md` (gitignored) —
**if it exists, read it at the start of the session** (Claude Code loads it
automatically).

## First command of every session

```sh
./shot status          # or --json
```

Shows the whole project state: inputs (analyzed?), selects with ★ ratings,
decision notes, speed variants and the montage target. **The manifest
`output/project.json` is the source of truth for decisions** (source ranges,
stars, "stays slow — mood") — do not reconstruct them from memory or chat history.

## CLI: `./shot <command>` (wrapper around `.venv/bin/python -m pipeline`)

Conventions (all commands): `--json` = result on stdout, logs always on stderr
(batch commands wrap results in `{"results": [...]}`, single-file ones return
a bare object); native batch (many files as arguments — **do not write shell
loops**); exit != 0 on error. Below is only an index — **take flag semantics
from `./shot <command> --help`**, not from memory; decision criteria from
`docs/decision-rules.md`; workflows from the skills.

```sh
./shot status           # project dashboard: inputs, selects, sequence, render, music, publishing
                      # (compact: analyzed inputs collapsed, long cut notes truncated — --full/--json for everything)
./shot scan FILE...     # smoothness analysis + contact.png in one pass (mtime cache);
                      # a batch runs files in parallel (--jobs N; 1 = serial)
./shot sheet FILE...    # contact sheet alone (scan already makes one — don't run after scan)
./shot frames FILE T... # 1280px evaluation frames -> output/<stem>/frames/
./shot jitter FILE      # settles jitter vs smooth maneuver within a range
./shot select FILE A B  # cut a select from the source + manifest entry (--label/--stars/--note);
                      # --plan PLAN.jsonl = batch of many selects, resumable (skips already-cut)
./shot pace ...         # screen pace (%/s) + recommendation; --profile = dull stretches (DULL) for shot trim
./shot speed CLIP N     # sped-up variant _xN (refuses on _x* files)
./shot tag CLIP...      # content tags + metadata (instead of editing the manifest by hand); --reject/--unreject
./shot trim CLIP A B    # trim a select: re-cut from the SOURCE, variants and pace refreshed automatically
./shot sequence ...     # montage order (the files ACTUALLY spliced) + lint; --target SEC;
                      # --note/--append-note = persistent cut decision note; no args: preview
./shot montage          # splice the sequence (crossfade = re-encode; --xfade 0 = stream-copy draft;
                      # --draft = fast preview encode with transitions; a fresh matching
                      # render is skipped (--force re-renders);
                      # --smooth = mixed-frame-rate interpolation; --files + --out = one-off
                      # ad-hoc render without touching the manifest)
./shot smooth [CLIP...] # warm the interpolation cache in the background BEFORE the final `shot montage --smooth`
./shot locate ...       # read-only: montage timeline <-> shots (TIME/FILE/full timeline; --files = external)
./shot grade ...        # color grading, NON-DESTRUCTIVE (baked at montage render): --analyze =
                      # color stats; FILE + correction flags = per-select corrections;
                      # --look/--look-lut = the cut's look; --source --input-lut = log
                      # normalize; --preview = before/after grid (inspect the image);
                      # no args: grade state; grade change => render stale
./shot music ...        # --generate = track from Stable Audio (COSTS MONEY — gates in decision-rules "Music");
                      # TRACK... = mux onto the render -> output/cuts/<cut>-final.mp4; --probe = analysis; no args: state
./shot publish ...      # YT thumbnail (--frame, inspect the resulting image) / title + description
                      # (--title --description-file); no args: publishing state
./shot ui               # read-only web UI for the human: selects gallery (thumbnails,
                      # ★/tags/notes, clip + source preview), cuts, inputs; localhost,
                      # blocks until Ctrl-C (--port N, 0 = auto; --no-open)
./shot archive NAME     # output/ -> archive/<date>_<name>/ + clean start; shot restore = the reverse
./shot config           # machine config: --input-dir (e.g. an SD card), --jobs N (worker cap
                      # for scan batch/probes/UI thumbnails; 1 = serial on weak machines),
                      # --hwaccel auto|off (VAAPI/VideoToolbox); no flags shows the state
./shot validate         # check manifest/summary/config files against the schema contract (pipeline/schemas/)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tskupinski/shotpilot](https://github.com/tskupinski/shotpilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
