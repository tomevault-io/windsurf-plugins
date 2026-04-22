---
trigger: always_on
description: This project is a static Game Boy recompiler. It turns ROMs into generated C plus a shared runtime.
---

## Project Overview
This project is a static Game Boy recompiler. It turns ROMs into generated C plus a shared runtime.

Important directories:

- `recompiler/`: source for `gbrecomp`
- `runtime/`: shared runtime library (`gbrt`) used by generated projects
- `roms/`: source ROMs used for testing and local experiments
- `output/`: generated ROM projects and multi-ROM launcher outputs
- `logs/`: runtime logs, interpreter logs, benchmark results, and repro captures
- `tools/`: helper scripts for testing, benchmarking, trace capture, and log analysis

Use `README.md` for current user-facing workflows, `TODO.md` for the live project backlog, and `GBC.md` for the current Game Boy Color support status / remaining work.

## Hardware References
- When implementing or debugging hardware behavior, consult `tech_docs/pan_docs.md` first.
- Use the local `SameBoy/` codebase as the second reference when Pan Docs is ambiguous or you need a proven implementation to compare against.
- This is especially important for CGB registers, timing, palette handling, HDMA, speed switching, and DMG-on-CGB compatibility behavior.

## Build Standards
- Always use CMake + Ninja.
- Run shell commands from the repo root when possible.
- Prefer absolute paths in tool output and references when useful, but keep shell commands readable.

Main project:

```bash
cmake -G Ninja -B build .
ninja -C build
```

## Required Sync Workflow
If you change recompiler logic, generated CLI/codegen behavior, runtime behavior, or runtime headers, keep the generated test project in sync.

Canonical sync flow:

```bash
ninja -C build
./build/bin/gbrecomp <checked-in test ROM> -o <checked-in generated test project>
cmake -G Ninja -S <checked-in generated test project> -B <checked-in generated test project>/build
ninja -C <checked-in generated test project>/build
```

Notes:

- Use the checked-in generated test project as the default smoke test.
- Put ad hoc generated projects under `output/`, not the repo root.
- Put logs under `logs/`, not next to binaries.
- If a generated project looks like it is ignoring a runtime change, rebuild that generated project explicitly. Stale generated builds can produce misleading results.

## Current State Worth Remembering
- Multi-ROM generation exists and emits a shared launcher project.
- The multi-ROM launcher is graphical by default and uses SDL + ImGui.
- Battery-backed save loading/saving is wired through the runtime and SDL platform layer.
- Interpreter fallback instrumentation exists and should be used before guessing about coverage/performance problems.
- Benchmarking support exists and should use the benchmark helper script, not ad hoc wall-clock timing of normal windowed builds.

## Accuracy Workflow
When working on correctness, use the runtime and tooling that already exist before inventing new debug paths.

Preferred tools:

- Differential mode for generated vs interpreter comparisons:
  ```bash
  ./output/game/build/game --differential 500000 --differential-log 100000
  ```
- Record and replay input for stable repro:
  ```bash
  ./output/game/build/game --log-file logs/session.log --record-input logs/session.input
  ./output/game/build/game --input "$(cat logs/session.input)"
  ```
- Use cycle-anchored recorded inputs when possible. `--record-input` already writes those by default.
- If you need screenshots or frame comparisons, use `--dump-frames`, `--dump-present-frames`, and the existing tools under `tools/`.

If debugging a game-specific issue, prefer keeping:

- the ROM in `roms/`
- the generated project in `output/<name>/`
- the log in `logs/<name>.log`
- the recorded input in `logs/<name>.input`

## Interpreter Fallback Workflow
Interpreter fallback is already instrumented. Use that signal.

Start here:

```bash
./output/game/build/game \
  --log-file logs/interpreter.log \
  --limit 500000 \
  --log-frame-fallbacks \
  --report-interpreter-hotspots \
  --interpreter-hotspot-limit 12
```

Then summarize with:

```bash
python3 tools/summarize_interpreter_log.py logs/interpreter.log
```

Use this workflow when:

- generated execution falls back unexpectedly
- performance is poor and you suspect interpreter use
- you need hotspot data to guide recompiler improvements

Important interpretation rule:

- if the summary says `No interpreter fallback recorded`, the remaining slowdown is not an interpreter-coverage problem

## Recompiler Debugging
If analysis hangs, misses code, or produces obviously bad output:

- trace analysis:
  ```bash
  ./build/bin/gbrecomp roms/game.gb --trace
  ```
- stop early to find where analysis goes off the rails:
  ```bash
  ./build/bin/gbrecomp roms/game.gb --limit 10000
  ```
- watch for `[ERROR] Undefined instruction` lines
- use `--add-entry-point` for known missed code entrypoints
- use `--no-scan` if aggressive scanning is misclassifying data as code
- use `--annotations <file>` when you have trusted function starts or ROM data ranges from a disassembly/project map

If you are working with imported symbols or generated names, remember that generated projects can emit `*_metadata.json` sidecars. Prefer using those instead of scraping generated C when a sidecar already answers the question.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcanite24/gb-recompiled](https://github.com/arcanite24/gb-recompiled) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
