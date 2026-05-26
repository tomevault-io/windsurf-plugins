---
trigger: always_on
description: This file is for coding agents working inside this repository. Human-facing positioning lives in `README.md`. Agent-facing priority is installation, repeatable runs, and not breaking query-trace semantics.
---

# SQL-ManyThing Agent Guide

This file is for coding agents working inside this repository. Human-facing positioning lives in `README.md`. Agent-facing priority is installation, repeatable runs, and not breaking query-trace semantics.

## Mental Model

SQL-ManyThing has three phases:

```text
Phase 1: build .srcidx/source.db with files + files_fts
Phase 2: add enrichment into the same DB
Phase 3: install sqlite wrapper + query log so agent queries are recorded and reused
```

Agents should keep query-time work in SQL. Do not switch between generic shell search and SQL unless setup is broken.

## Repository Layout

```text
scripts/
├── phase1/manything_build_db.py
├── phase2/enrich_cymbal.py
├── phase2/enrich_graphify.py
├── phase2/enrich_java_build.py
├── phase2/uht_enrich.py
├── phase3/manything_query_log.py
├── phase3/sqlite3_wrapper.sh
├── phase3/SQL-ManyThing-query-log
└── verify/verify_ue_uht_sql.py

references/
├── phase1/
├── phase2/
├── phase3/
├── platforms/
└── unreal/
```

## Install Phase 3 Locally

Run from repo root:

```bash
python3 scripts/phase3/manything_query_log.py init
mkdir -p ~/.local/bin
cp scripts/phase3/sqlite3_wrapper.sh ~/.local/bin/sqlite3
cp scripts/phase3/SQL-ManyThing-query-log ~/.local/bin/SQL-ManyThing-query-log
chmod +x ~/.local/bin/sqlite3 ~/.local/bin/SQL-ManyThing-query-log
```

Ensure `~/.local/bin` is before `/usr/bin`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Register aliases in exactly this format:

```bash
echo 'MANYTHING_myproject="/path/to/project"' >> ~/.hermes/manything/aliases.sh
```

The wrapper expects `MANYTHING_<project>`, not `SQLMANYTHING_<project>`.

## Verify Phase 3

Do not use `ls /manything/<project>/source.db`. `/manything/...` is not a filesystem path. It is a sqlite wrapper virtual path.

Correct verification:

```bash
sqlite3 /manything/myproject/source.db "SELECT COUNT(*) FROM files"
SQL-ManyThing-query-log import
sqlite3 :trace "SELECT COUNT(*) FROM query_log"
sqlite3 :trace ".tables"
```

Expected trace tables:

```text
query_log query_notes query_trace
```

`query_trace` does not belong inside any project `.srcidx/source.db`.

## Build Phase 1

General git repo:

```bash
python3 scripts/phase1/manything_build_db.py /path/to/project --git --ext .ts,.tsx,.js,.jsx,.json,.md
```

Directory with explicit `.gitignore`:

```bash
python3 scripts/phase1/manything_build_db.py /path/to/project --gitignore /path/to/project/.gitignore
```

Installed Unreal Engine core profile:

```bash
python3 scripts/phase1/manything_build_db.py /path/to/Engine \
  --gitignore /path/to/Engine/.gitignore \
  --profile unreal-installed-core
```

Profile behavior:

```text
unreal-installed-core:
  keep .h,.cpp,.cs,.usf,.ush,.hlsl,.py,.ini,.uplugin
  skip ThirdParty/Content/Platforms/ScriptModules high-noise paths

unreal-installed-full:
  keep broad source/data extensions
  intended only for deep ThirdParty/data exploration
```

## Windows / WSL Guidance

When indexing a large Windows-hosted tree, prefer Windows Python for Phase 1. WSL DrvFs writes are slow.

Template:

```text
templates/run_phase1_unreal_windows.bat
```

Copy these into a Windows-side run directory:

```text
run_phase1_unreal_windows.bat
manything_build_db.py
.gitignore
```

Then run the BAT from Windows.

WSL can still query the resulting DB:

```bash
sqlite3 /mnt/d/Path/To/Engine/.srcidx/source.db "SELECT COUNT(*) FROM files"
```

## Phase 2 Enrichment

cymbal symbols:

```bash
python3 scripts/phase2/enrich_cymbal.py /path/to/project
```

graph/document enrichment:

```bash
python3 scripts/phase2/enrich_graphify.py /path/to/project
```

Java build enrichment:

```bash
python3 scripts/phase2/enrich_java_build.py /path/to/project
```

Unreal Installed Build UHT enrichment:

```bash
python3 scripts/phase2/uht_enrich.py \
  --db /path/to/Engine/.srcidx/source.db \
  --uht-dir /path/to/Engine/Intermediate/Build/Win64/UnrealEditor/Inc \
  --source-prefix Engine/ \
  --batch 500
```

For installed Unreal Engine, run UHT enrich as the primary Phase 2 path. Do not use cymbal/graphify as the main Unreal reflection strategy.

## Query-Time Rules for Agents

Before first query:

```bash
SQL-ManyThing-query-log import
sqlite3 :trace "
WITH intent(term) AS (
  VALUES ('files'), ('ext'), ('path'), ('symbols'), ('file_enrich'),
         ('graph'), ('README'), ('package'), ('src')
)
SELECT id, project, tag, note, substr(sql_text, 1, 180) AS sql_preview
FROM query_trace
WHERE project='<project>'
  AND (tag IS NOT NULL OR EXISTS (
    SELECT 1 FROM intent WHERE lower(sql_text) LIKE '%' || lower(term) || '%'
  ))
ORDER BY tag IS NULL, id DESC
LIMIT 12;"
```

Expand natural-language intent into SQL-facing terms using world knowledge. For example, "implementation overview" should search terms like:

```text
files, ext, path, symbols, file_enrich, graph, README, package, src, layout, prepare, measure, benchmark
```

If a trace row is useful, tag it with SQL, not a separate CLI command:

```bash
sqlite3 :trace "
INSERT INTO query_notes (log_id, note, tag, created_at)
VALUES (<id>, '<reuse note>', 'useful_pattern', strftime('%s','now'));
"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IOchair/SQL-ManyThing](https://github.com/IOchair/SQL-ManyThing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
