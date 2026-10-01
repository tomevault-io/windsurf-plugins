---
trigger: always_on
description: Keep **docs** and **code** separate. Skill instructions (`SKILL.md`) stay with the skill definition; executable scripts live at the repo root under `.ai/`, named after the skill.
---

# Skill Dependency Manager

## Skill layout

Keep **docs** and **code** separate. Skill instructions (`SKILL.md`) stay with the skill definition; executable scripts live at the repo root under `.ai/`, named after the skill.

| What | Where |
|------|-------|
| Skill instructions (`SKILL.md`, references) | The skill's own folder (next to `SKILL.md`) |
| Skill scripts (Python / shell / etc.) | `.ai/<skill-name>/` |
| Script tests (`test_<script>.py`) | `.ai/<skill-name>/` — default `python` skills only |
| CLI manual (`<skill-name>.md`) | `cli/` — copy-paste unit-test and manual CLI commands |
| Toolchains (Python, FFmpeg, venvs) | `.dependency/` |

```
project-root/
├── cli/
│   ├── audio-to-wav.md           # unit tests + manual CLI
│   └── ai-text-to-speech.md
├── .ai/
│   ├── audio-to-wav/
│   │   ├── convert.py
│   │   └── test_convert.py
│   └── ai-text-to-speech/
│       └── tts.py
└── .dependency/
```

- The folder name under `.ai/` **must match** the skill name.
- Put scripts **directly** under `.ai/<skill-name>/` — do not add an extra `scripts/` directory.
- **Default `python` skills:** include a test file named after the script (`convert.py` → `test_convert.py`). If a directory has several scripts, give each one a matching `test_<script>.py`.
- **Every skill with scripts** gets `cli/<skill-name>.md` — unit-test commands (when present) and manual CLI examples using the correct manifest `bin`. Write each command as **one line** (no `\` line breaks) so it pastes into the console as a single command.
- **Non-default runtime** (a tool venv or `python-3.11`, etc.): do **not** add `test_*.py`. Put CLI commands in `cli/<skill-name>.md` only.
- Do **not** put executable scripts next to `SKILL.md`.
- SKILL.md commands must point at `.ai/<skill-name>/...`.
- Every skill script must start with a module docstring that includes **Usage**: full commands from the repo root, using the manifest interpreter. Write each command as **one line** (no `\` line breaks). Never use host `python`.
- If the skill uses the **default** `python` entry, say so in the docstring.
- If the skill does **not** use default Python, the docstring **must** say that explicitly: which manifest entry, which version (e.g. Python 3.11), and that default `python` must not be used.

Default `python`:

```
"""
Short description.

Run through default python from .dependency/manifest.json.
Never use host python/py.

Usage
-----
    .dependency/python/python .ai/<skill-name>/<script>.py input [flags]
"""
```

Non-default runtime (must state version + entry):

```
"""
Short description.

Not default python. Run through the <entry> manifest bin
(Python 3.11 venv at .dependency/<entry>/.venv/).
Never use default python or host python/py.

Usage
-----
    .dependency/<entry>/.venv/Scripts/python.exe .ai/<skill-name>/<script>.py input [flags]
"""
```

## Python style (`.ai/` skill scripts)

- **One-line imports.** Each `import` or `from … import` must stay on a **single line** — do not wrap imports in parentheses across multiple lines.

## Run skill scripts

When a skill has scripts, run them from the project root as the skill docs say. Do not use your own commands unless the skill says the script is for reference only. Canonical script path: `.ai/<skill-name>/`.

Run tests the same way — same interpreter from `.dependency/manifest.json`, from the repo root:

```bash
.dependency/python/python.exe .ai/audio-to-wav/test_convert.py
```

If a skill uses a **non-default** runtime, skip `test_*.py`. Use the CLI in `cli/<skill-name>.md` (that runtime's `bin`).

Manual CLI examples for any skill live in `cli/<skill-name>.md`, not under `.ai/`.

Do not use host `python` / `pytest` to run skill tests.

### No bypass

Even when a skill script wraps FFmpeg or another CLI, call it through the skill script — do not hand-write equivalent commands.

### Workflow

1. Find the script and command in the skill docs.
2. Run it. If something is missing, install it (see **Dependencies** below or skill setup steps), then run the same command again.
3. If it fails, fix the setup or inputs and try again. Ask before using a different approach.

After installing anything, say what you installed and which command you ran.

## Skill pipelines

When the user lists **multiple skills in sequence**, chain them: **each step's output is the next step's input**. Never point a later step back at the original source.

Each skill writes beside its input into `<input-dir>/<skill-name>/`, so chained runs nest:

```
source/file.ext
  → source/skill-a/out.ext
  → source/skill-a/skill-b/out.ext
  → source/skill-a/skill-b/skill-c/out.ext   ← final
```

1. **Order** — user's list, left to right (or top to bottom).
2. **Input** — step 1 uses the source file; later steps pass the **prior output** using that skill's input flag from `SKILL.md` (`--image`, `--audio`, `--video`, etc.).
3. **One file per run** — one input per invocation; repeat the full pipeline for each file when batching a folder.
4. **No bypass** — run every step through its bundled script; do not merge steps into one hand-written command.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [godot-fun/gai](https://github.com/godot-fun/gai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
