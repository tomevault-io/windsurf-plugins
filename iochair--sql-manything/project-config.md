---
trigger: always_on
description: A* source-code search over SQL-ManyThing SQLite index. Four canonical SQL templates (DISCOVER→TRACE_DEPS→EXTRACT→EXTRACT_BLOCK) + Abstraction Frame with budget tracking. No substr guessing, no whole-file reads.
---


# SQL-ManyThing

Indexes a source tree into a SQLite DB. Query model: Abstraction Frame → DISCOVER → EXTRACT → EXTRACT_BLOCK, with optional TRACE_DEPS for candidate disambiguation.

**Reading order is execution order.** Start at §0, follow steps in sequence. Do not jump to SQL templates before writing a Frame.

## §0 When to Use

**Use when:**
- A project has `.srcidx/source.db` (if not, see §1.3 to create one). Query via `/manything/<project>/source.db`.
- The user asks for code search, tracing, implementation lookup, or architecture inspection.
- You need precise, offset-based extraction without whole-file reads.

**Do NOT use when:**
- No index exists for the project.
- You're looking at <100 files total — `code_search` / `code_extract` is faster.
- The target is logs, config, markdown, or non-code text — use `search_files` / `read_file`.
- You need a single function from a known file — `code_extract` is 1 call.

## §1 Pre-flight — Know Your DB Before You Query

Before ANY query, check what tables are available. Not every project is fully enriched.

### 1.1 Schema check

```bash
sqlite3 /manything/<project>/source.db "
SELECT name FROM sqlite_master
WHERE type='table' AND name IN ('files','files_fts','v_enriched','enrich_file_deps','enrich_file_refs','enrich_depth_segments')
ORDER BY name;"
```

| Tables present | Capabilities |
|---|---|
| `files` + `files_fts` only | FTS5 search only. No v_enriched — skip EXTRACT/EXTRACT_BLOCK. Use `files.content` or fall back to `read_file`. |
| + `v_enriched` | Full DISCOVER → EXTRACT → EXTRACT_BLOCK chain. |
| + `enrich_file_deps` | TRACE_DEPS available. |
| + `enrich_depth_segments` | `block_content_full` + `scope_end_offset` for complete body extraction. |
| + `enrich_file_refs` (no deps) | Raw #include/import strings — query refs table directly. |

### 1.2 File count

```bash
sqlite3 /manything/<project>/source.db "SELECT count(*) AS file_count FROM files;"
```

| File count | Strategy |
|---|---|
| <10K | Fast LIKE scans OK |
| 10K–50K | Prefer FTS5 for discovery; narrow LIKE with `file_path` |
| >50K | FTS5 only for discovery; NEVER run unfiltered `block_content LIKE '%X%'` without `file_path` |

### 1.3 Setup: Phase 1 → 2 → 3

A fully enriched project goes through three phases. After Phase 1 each phase is optional — stop when you have the tables you need (§1.1).

**Phase 1 — Create the index** (`references/phase1/phase1-setup.md`):
```bash
python3 scripts/phase1/manything_build_db.py /path/to/project
```
This creates `.srcidx/source.db` with `files` + `files_fts`. Enough for FTS5-only search.

**Phase 2 — Enrich with structure + dependencies** (`references/phase2/enrich-covercheck-workflow.md`):
```bash
# Universal workflow (all languages, always applicable):
python3 scripts/phase2/enrich_depth_segments.py /path/to/project --batch 500
python3 scripts/phase2/enrich_file_refs.py       /path/to/project --batch 500
python3 scripts/phase2/flatten_file_deps.py      /path/to/project
python3 scripts/phase2/create_enriched_view.py   /path/to/project
```
This adds `v_enriched`, `enrich_file_deps`, `enrich_file_refs`, `enrich_depth_segments` — the full DISCOVER→EXTRACT→EXTRACT_BLOCK chain. If re-running on a previously enriched DB, see `references/old-format-cleanup.md`.

Optional enrichments (not needed for most queries):
- `enrich_cymbal.py` — symbol definitions via cymbal CLI (Python/Go/JS only; needs `cymbal` binary)
- `enrich_graphify.py` — AST graph for Python + Markdown (does not cover TSX/JSX)
- `enrich_java_build.py` — Java-specific

On Windows, `scripts/phase2/run_phase2_universal_windows.bat` runs all 4 steps. On Linux/WSL/macOS, run the Python scripts directly as shown above.

**Phase 3 — Install the sqlite3 wrapper** (enables `/manything/` paths + `:trace`):
```bash
# Cross-platform (Windows/Linux/macOS):
python3 scripts/phase3/install.py

# Or bash-only (Linux/WSL/macOS):
./install.sh
```
This places a `sqlite3` wrapper in `~/.local/bin/` that intercepts `/manything/<project>/source.db` and `:trace`. On Windows, `install.py` also copies `sqlite3-real.exe` and creates a `.cmd` wrapper. Ensure `~/.local/bin` is before `/usr/bin` in PATH.

After Phase 3, register the project:
```bash
echo 'MANYTHING_<project>="/path/to/project"' >> ~/.hermes/manything/aliases.sh
```
Then proceed with §1.1 schema check.

See `references/phase3/phase3-design-rationale.md` for architecture details.

### 1.4 Trace — search past queries (`:trace`)

The sqlite3 wrapper logs every `/manything/` query to `~/.hermes/manything/query_log.db`. Use `:trace` as a virtual database name to search past queries — it resolves to the global query log automatically.

**Search by project + keyword:**
```bash
sqlite3 :trace "
SELECT id, sql_text, tag FROM query_trace
WHERE project = '<project>'
  AND sql_text LIKE '%<keyword>%'
ORDER BY id DESC LIMIT 10;"
```

**Tag a useful query pattern** (agent discovers a reusable SQL shape):
```sql
INSERT INTO query_notes(log_id, note, tag, created_at)
VALUES (<id>, 'brief description of the pattern', '<short_tag>', strftime('%s','now'));
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IOchair/SQL-ManyThing](https://github.com/IOchair/SQL-ManyThing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
