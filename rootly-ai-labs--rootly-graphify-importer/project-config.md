---
trigger: always_on
description: This project turns Rootly incident data into a knowledge graph. The Rootly corpus lives in `graphify-rootly-data/` and the graph output in `graphify-out/`.
---

# rootly-graphify

This project turns Rootly incident data into a knowledge graph. The Rootly corpus lives in `graphify-rootly-data/` and the graph output in `graphify-out/`.

**Always-on rules:**
- Before answering incident or architecture questions, read `graphify-out/GRAPH_REPORT.md` for god nodes and community structure
- If `graphify-out/wiki/index.md` exists, navigate it instead of reading raw files
- After modifying code files, run: `python -c "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"` to keep the graph current

---

# graphify pipeline — full instructions

When the user asks you to run graphify on a path (e.g. "run graphify on graphify-rootly-data"), follow these steps in order.

## Usage patterns

```
graphify <path>                  # full pipeline
graphify <path> --mode deep      # richer INFERRED edges
graphify <path> --update         # re-extract only new/changed files
graphify query "<question>"      # BFS traversal of graph.json
graphify path "NodeA" "NodeB"    # shortest path between two nodes
graphify explain "NodeName"      # explain a single node
```

---

## Step 1 — Ensure graphify is installed

```bash
python -c "import graphify" 2>/dev/null || pip install graphifyy -q
```

---

## Step 2 — Detect files

```bash
python -c "
import json
from graphify.detect import detect
from pathlib import Path
result = detect(Path('INPUT_PATH'))
Path('.graphify_detect.json').write_text(json.dumps(result, indent=2), encoding='utf-8')
print(result['total_files'], 'files,', result['total_words'], 'words')
for k, v in result.get('files', {}).items():
    if v: print(f'  {k}: {len(v)} files')
"
```

Replace INPUT_PATH with the actual path. Present a clean summary:
```
Corpus: X files · ~Y words
  docs:   N files
  code:   N files
```

- If `total_files` is 0: stop with "No supported files found."
- If `total_words` > 2,000,000 OR `total_files` > 500: warn and ask which subfolder to run on.
- Otherwise: proceed directly to Step 3.

---

## Step 3 — Extract entities and relationships

This step has two parts: **AST extraction** (code, deterministic) and **semantic extraction** (LLM, costs tokens). Run both in parallel.

### Part A — AST extraction (code files)

```bash
python -c "
import sys, json
from graphify.extract import collect_files, extract
from pathlib import Path

detect = json.loads(Path('.graphify_detect.json').read_text(encoding='utf-8'))
code_files = []
for f in detect.get('files', {}).get('code', []):
    code_files.extend(collect_files(Path(f)) if Path(f).is_dir() else [Path(f)])

if code_files:
    result = extract(code_files)
    Path('.graphify_ast.json').write_text(json.dumps(result, indent=2), encoding='utf-8')
    print(f'AST: {len(result[\"nodes\"])} nodes, {len(result[\"edges\"])} edges')
else:
    Path('.graphify_ast.json').write_text(json.dumps({'nodes':[],'edges':[],'input_tokens':0,'output_tokens':0}), encoding='utf-8')
    print('No code files - skipping AST')
"
```

### Part B — Semantic extraction (parallel subagents)

**MANDATORY: Use parallel subagents here. Reading files one-by-one is 5-10x slower.**

#### B0 — Check cache

```bash
python -c "
import json
from graphify.cache import check_semantic_cache
from pathlib import Path

detect = json.loads(Path('.graphify_detect.json').read_text(encoding='utf-8'))
all_files = [f for files in detect['files'].values() for f in files]
cached_nodes, cached_edges, cached_hyperedges, uncached = check_semantic_cache(all_files)

if cached_nodes or cached_edges or cached_hyperedges:
    Path('.graphify_cached.json').write_text(json.dumps({'nodes': cached_nodes, 'edges': cached_edges, 'hyperedges': cached_hyperedges}), encoding='utf-8')
Path('.graphify_uncached.txt').write_text('\n'.join(uncached), encoding='utf-8')
print(f'Cache: {len(all_files)-len(uncached)} hit, {len(uncached)} need extraction')
"
```

#### B1 — Split uncached files into chunks of 20-25

Load `.graphify_uncached.txt`, split into chunks. Each image gets its own chunk.

#### B2 — Dispatch ALL subagents in a single message (parallel)

Launch one subagent per chunk simultaneously. Each subagent receives this prompt:

```
You are a graphify extraction subagent. Read the files listed and extract a knowledge graph fragment.
Output ONLY valid JSON matching the schema below - no explanation, no markdown fences, no preamble.

Files (chunk CHUNK_NUM of TOTAL_CHUNKS):
FILE_LIST

Rules:
- EXTRACTED: relationship explicit in source (import, call, citation, "see §3.2")
- INFERRED: reasonable inference (shared data structure, implied dependency)
- AMBIGUOUS: uncertain - flag for review, do not omit

Doc/paper files: extract named concepts, entities, citations. Also extract rationale — WHY decisions were made. These become nodes with `rationale_for` edges.
Image files: use vision — understand what the image IS (UI layout, chart trend, diagram components).
DEEP_MODE (if --mode deep): be aggressive with INFERRED edges. Mark uncertain ones AMBIGUOUS.

Semantic similarity: if two concepts solve the same problem without a structural link, add a `semantically_similar_to` edge (INFERRED, confidence 0.6-0.95). Only when non-obvious.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rootly-AI-Labs/rootly-graphify-importer](https://github.com/Rootly-AI-Labs/rootly-graphify-importer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
