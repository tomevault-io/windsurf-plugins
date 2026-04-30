---
trigger: always_on
description: > Guidelines for AI coding agents working in this bio-inspired neural network codebase.
---

# AGENTS.md — bio_inspired_nanochat

> Guidelines for AI coding agents working in this bio-inspired neural network codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

### Python Toolchain

- Use **uv** for everything Python.
  - ❌ Never use `pip`, `pip install`, or `pip freeze`.
  - Lockfiles: only `uv.lock`. Do not introduce any other lockfile.
- Target **Python 3.14** only. No need to support older Python versions.
- Use **pyproject.toml** exclusively for project configuration. Never create `requirements.txt`.
- Virtual environment is managed by uv (`.venv/`).

Run scripts via:
```bash
uv run python -m scripts.base_train
uv run python -m pytest tests/
```

Add dependencies:
```bash
uv add <package>
```

---

### Environment Configuration

We load all configuration from the existing `.env` file. We NEVER use `os.getenv()` or `dotenv` directly. Use **python-decouple** in this exact pattern:

```python
from decouple import Config as DecoupleConfig, RepositoryEnv

# Initialize decouple config (project-local .env; must not be overwritten)
decouple_config = DecoupleConfig(RepositoryEnv(".env"))

# Configuration
SOME_VAR = decouple_config("SOME_VAR", default="default_value")
```

The `.env` file exists but may not be visible to you. **NEVER overwrite it**.

---

### Code Quality Checks

**CRITICAL:** After any substantive code changes, you MUST verify no type or lint errors were introduced:

```bash
# Lint check (auto-fix where safe)
uv run ruff check --fix

# Type check
uv run ty check
```

If errors appear, carefully understand and fix each one. Read sufficient context to understand the RIGHT way to fix them, not just silence them.

---

### Code Editing Discipline

- Do **not** run scripts that bulk-modify code (codemods, invented one-off scripts, giant `sed`/regex refactors).
- Large mechanical changes: break into smaller, explicit edits and review diffs.
- Subtle/complex changes: edit by hand, file-by-file, with careful reasoning.

---

### Backwards Compatibility & File Sprawl

We optimize for a clean architecture now, not backwards compatibility.

- No "compat shims" or "v2" file clones.
- When changing behavior, migrate callers and remove old code **inside the same file**.
- New files are only for genuinely new domains that don't fit existing modules.
- The bar for adding files is very high.

---

### Logging & Console Output

- Use the **rich** library for all console output (informative, detailed, colorful).
- Prefer structured logging via Python's `logging` module over raw `print()`.
- No random print statements in library code; if needed, make them dev-only and clean them up.
- Log structured context: IDs, step numbers, metrics, etc.
- If a logger helper exists (e.g., in `common.py`), you must use it; do not invent a different pattern.

---

### Third-Party Libraries

When unsure of an API, look up current docs (late-2025) rather than guessing.

Key libraries in this project:
- **PyTorch** (torch) - Neural network framework
- **Triton** - GPU kernel compilation
- **maturin/PyO3** - Rust extensions (rustbpe tokenizer)
- **rich** - Console output
- **wandb** - Experiment tracking
- **tensorboard** - Training visualization

---

## MCP Agent Mail — Multi-Agent Coordination


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/bio_inspired_nanochat](https://github.com/Dicklesworthstone/bio_inspired_nanochat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
