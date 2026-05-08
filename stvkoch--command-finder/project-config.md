---
trigger: always_on
description: Guidance for Claude Code agents working in this repository. Read this before editing code or adding command entries.
---

# CLAUDE.md — Command Finder (`cf`)

Guidance for Claude Code agents working in this repository. Read this before editing code or adding command entries.

## What this project is

`cf` is a natural-language shell command finder. A user types `cf "find large files older than 30 days"` and gets an interactive picker of real shell commands. The selected command is injected into the zsh readline buffer, copied to the clipboard, printed, or sent to tmux.

Under the hood:

1. `sentence-transformers` (`all-MiniLM-L6-v2`, 384-dim) encodes the query.
2. `sqlite-vec` performs cosine-distance vector search over indexed **patterns**.
3. `simple-term-menu` renders an interactive selector.
4. `shell/cf.zsh` pushes the chosen command onto the zsh edit buffer via `print -z`.

## Project layout

```
src/cf/
├── cli.py              # Typer entry point (cf ...)
├── config.py           # Env vars, paths, model name (CF_TOP, CF_MODEL, CF_DB_DIR, CF_DATA_DIR)
├── db.py               # SQLite + sqlite-vec schema and batch inserts
├── embeddings.py       # sentence-transformers / ONNX wrapper
├── search.py           # Vector search + dedup
├── seed.py             # Reads data/commands/*.json and populates the DB
├── selector.py         # Interactive terminal menu
├── output.py           # stdout / clipboard / tmux output handlers
└── data/commands/      # JSON seed files — the source of truth for commands
```

- DB lives at `$CF_DB_DIR/cf.db` (default `~/.local/share/cf/cf.db`).
- Seed JSON files live at `src/cf/data/commands/*.json` and ship with the package (`pyproject.toml` → `package-data`).

## How seeding works

`cf --seed` (or `cf --seed --force` to wipe first) runs `src/cf/seed.py → seed_database()`. It:

1. **Loads** every `*.json` under `DATA_DIR` via `load_seed_files()`.
2. **Flattens** them into three ordered lists:
   - `cmd_rows` — `(name, category, synopsis, description)`, deduped by `(category, name)`.
   - `pat_meta` — `(cmd_index, type, text, command_template, explanation)`.
3. **Encodes** every `pattern.text` in a single batch via `embeddings.encode_batch()` (cached — safe to re-run).
4. **Bulk-inserts** into SQLite using `bulk_load_pragmas` for speed, in this order:
   - `insert_commands_batch` → returns auto-generated `cmd_ids`.
   - `insert_patterns_batch` with resolved `command_id` → returns `pat_ids`.
   - `insert_embeddings_batch` into the `vec0` virtual table, keyed by `pattern_id`.
5. **Restores** safe PRAGMAs and commits.

### DB schema (`db.py::init_db`)

- `commands(id, name, category, synopsis, description)`
- `patterns(id, command_id FK, pattern_type, text, command_template, explanation)`
- `pattern_embeddings` — virtual `vec0(pattern_id PRIMARY KEY, embedding FLOAT[384])`
- `query_cache(query_text PK, embedding BLOB)` — memoizes encoded user queries.

Each **pattern** gets its own embedding. The search is pattern-level, not command-level — that's why good recall depends on writing multiple phrasings of the same task.

## Adding database entries (the common task for agents)

The database is **derived** from `src/cf/data/commands/*.json`. Never hand-insert rows into SQLite — edit the JSON and reseed.

### JSON file shape

One category per file. Filename should match the `category` field (e.g. `git.json` → `"category": "git"`).

```json
{
  "category": "git",
  "commands": [
    {
      "name": "git commit",
      "synopsis": "git commit [options]",
      "description": "Record changes to the repository",
      "patterns": [
        {
          "type": "example",
          "text": "commit staged changes with a message",
          "command": "git commit -m \"message\"",
          "explanation": "Records all staged changes with the given commit message"
        }
      ]
    }
  ]
}
```

### Field rules

- **`category`** — must match the surrounding file; used as the grouping key.
- **`name`** — canonical command name (`"git commit"`, `"tar"`, `"docker run"`). `(category, name)` is the dedup key in `seed.py`; duplicates collapse into one `commands` row but their patterns merge.
- **`synopsis`** — man-page style usage line. One line.
- **`description`** — one-sentence purpose. Shown in the selector preview.
- **`patterns[]`** — **this is what gets searched**. Each entry is one embedding.
  - **`type`** — currently always `"example"`. Keep it unless you have a reason.
  - **`text`** — natural-language description of *what the user wants to do*. Write how a user would phrase the task, not how the command works. This is the embedded string.
  - **`command`** — the exact shell command that will be injected into the user's prompt. Must be runnable; use placeholders like `<file>`, `/path/to/dir`, `name` where appropriate.
  - **`explanation`** — one-line explanation shown in the selector preview. Optional but recommended.

### Writing good patterns

- **Multiple phrasings per use case.** Users say "delete empty folders", "remove empty dirs", "clean up empty directories" — add all three as separate patterns with the same `command`. More patterns = better recall.
- **Phrase from the user's intent**, not the flag. ✅ "compress a directory into a tar.gz" — ❌ "tar -czf flag".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stvkoch/Command-Finder](https://github.com/stvkoch/Command-Finder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
