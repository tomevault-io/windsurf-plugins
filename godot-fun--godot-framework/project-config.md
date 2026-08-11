---
trigger: always_on
description: Skill dependency manager — run skill scripts, install missing tools into .dependency/
---


# Skill Dependency Manager

## Run skill scripts

When a skill has scripts, run them from the project root as the skill docs say. Do not use your own commands unless the skill says the script is for reference only.

### No bypass

Even when a skill script wraps FFmpeg or another CLI, call it through the skill script — do not hand-write equivalent commands.

### Workflow

1. Find the script and command in the skill docs.
2. Run it. If something is missing, install it (see **Dependencies** below or skill setup steps), then run the same command again.
3. If it fails, fix the setup or inputs and try again. Ask before using a different approach.

After installing anything, say what you installed and which command you ran.

## Dependencies

External CLIs, language runtimes, and skill-only toolchains install into `.dependency/`. Do not put **project/business** packages here (Godot addons, game server deps, app `requirements.txt`, etc.).

| Kind | Name examples | Install location |
|------|---------------|------------------|
| Language runtime | `python`, `python-3.11`, `node-20`, `rust-1.75`, `go-1.22` | `.dependency/<name>/` |
| CLI tool (standalone binary) | `ffmpeg`, `gemini-watermark`, `git`, `jq`, `curl`, `imagemagick` | `.dependency/<name>/` or `.dependency/<name>-tool/` |
| Python third-party tool | `rembg`, cloned GitHub projects | `.dependency/<name>/.venv/` |

**Root:** `.dependency/`  
**Manifest:** `.dependency/manifest.json`

Each manifest name maps to a dedicated directory under `.dependency/`. After populating, set `populated: true` and correct `bin` paths in the manifest.

`populated: false` in `.dependency/manifest.json` is not a reason to skip a skill script. Install the missing tool here first, then run the same skill command again.

### Language runtimes — `.dependency/` only

**Never** use a language runtime from outside `.dependency/`.

This applies to every runtime in the table above (`python`, `python-3.11`, `node-20`, `rust-1.75`, `go-1.22`, etc.).

**Forbidden** — do not invoke, discover, or fall back to any of these:

- Commands on the host PATH or version managers — e.g. `python`, `py`, `python3`, `node`, `npm`, `cargo`, `go`, `pyenv`, `nvm`, `rustup`
- **Any absolute path** to an interpreter outside `.dependency/` — e.g. `C:\Python314\python.exe`, `/usr/bin/python3`, `~/miniconda3/python`
- **Host virtual environments** anywhere on disk — `.venv/`, `venv/`, `env/` under the user profile, other repos, Desktop, Downloads, `AppData`, `Program Files`, `~/.local`, etc.
- **Conda / Miniconda / Anaconda** base or named envs
- **IDE- or editor-bundled** runtimes (Cursor, VS Code, PyCharm, etc.)
- **Other projects'** Python/Node installs, even if they already have the package you need

The **only** allowed interpreter paths are `bin` values in `.dependency/manifest.json` (which must live under `.dependency/`). For Python third-party tools, that includes venvs at `.dependency/<tool-name>/.venv/` only.

- Resolve the runtime from `manifest.json` → `<entry>.bin`.
- If `populated: false`, install that runtime under `.dependency/` first — do **not** fall back to any host install.
- Skill docs may show shorthand (`python`, `py`, `node`); always substitute the manifest `bin` path when running commands.
- If a command fails, fix the `.dependency/` install — never retry with a different host Python or venv.

### Python default version

When a skill does **not** specify a Python version, assume **Python 3.14** as the default runtime.

- Install to `.dependency/python/` and register as the `python` entry in `manifest.json`.
- Skills that only reference `python` (no version suffix) rely on this default.
- If a skill explicitly requires another version (e.g. `python-3.11`), use a separate manifest entry and install directory instead.

The `python` runtime is for **stdlib-only** skill scripts (e.g. audio wrappers, path/batch wrappers). Do **not** `pip install` into `.dependency/python/` itself.

### Standalone CLI tools

When a skill depends on a portable upstream binary (e.g. FFmpeg), follow this order:

1. **Create the install directory** under `.dependency/` — use the manifest key name (e.g. `.dependency/ffmpeg/`).

2. **Download the release asset** from the upstream project (GitHub Releases, vendor site). Extract so `bin` in `manifest.json` points at the executable.

   ```bash
   # FFmpeg (Windows)
   # Download ffmpeg-release-essentials.zip → extract to .dependency/ffmpeg/
   # Result: .dependency/ffmpeg/bin/ffmpeg.exe
   ```

   On Unix, use the platform binary without `.exe` (e.g. `.dependency/ffmpeg/bin/ffmpeg`).

3. **Register** in `manifest.json` with `bin` pointing at the executable:

   ```json
   "ffmpeg": {
     "populated": true,
     "bin": ".dependency/ffmpeg/bin/ffmpeg.exe"
   }
   ```

4. **Run** through the skill script when one exists — do not hand-write equivalent CLI unless the skill says the script is reference-only:

   ```bash
   .dependency/python/python.exe .cursor/skills/audio-to-wav/scripts/convert.py audio/input.flac
   ```

   Some skills wrap the binary with a stdlib Python script (`python` manifest entry); others invoke the CLI binary directly via its own manifest entry (`ffmpeg`).

### Python third-party tools (`.venv`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [godot-fun/godot-framework](https://github.com/godot-fun/godot-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
