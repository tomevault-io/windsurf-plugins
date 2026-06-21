---
trigger: always_on
description: >
---


# Slicer Skill

This repository contains the information and helper scripts needed by an AI coding agent ("skill")
that is designed to answer questions about the [3D Slicer](https://www.slicer.org) application and
its extension ecosystem.  It is intentionally generic so that it can be consumed by any tool that
understands the SKILLS.md convention (e.g. Claude Code, OpenAI agents, etc.).

## Setup Modes

The setup script (`./setup.sh`) has two orthogonal axes:

1. **mode** — *what gets cloned* (full / lightweight / web)
2. **indexes** — *what ranked-search indexes get built* (none / bm25 / hybrid)

On first run, `setup.sh` prompts for both. Subsequent runs reuse the previous
choices, recorded in `.setup-stamp.json` as the `"mode"` and `"indexes"` fields.
You can override interactively or via flags.

### Mode (which clones)

| Mode          | Disk    | Setup time | What's local                                         |
| ------------- | ------- | ---------- | ---------------------------------------------------- |
| **full**      | ~15 GB  | ~20 min    | Source, deps, all 200+ extensions, discourse archive  |
| **lightweight**| ~1 GB  | ~2 min     | Source + ExtensionsIndex metadata (JSON only)         |
| **web**       | minimal | instant    | Nothing cloned — all access via web APIs              |

Override: `./setup.sh --mode full|lightweight|web`

### Indexes (which ranked-search backends)

| Indexes  | Disk    | Setup time                                  | What it gives you                                        |
| -------- | ------- | ------------------------------------------- | -------------------------------------------------------- |
| **none** | 0       | instant                                     | No ranked search; the agent uses grep/find/web APIs.     |
| **bm25** | ~65 MB  | ~15 s (foreground)                          | Lexical search (BM25) over slicer-source + slicer-discourse. Best for exact identifiers, symbols, paths. |
| **hybrid**| ~300 MB| ~15 s up front, ~20 min in the **background** | bm25 PLUS dense embeddings (sentence-transformers/all-MiniLM-L6-v2). Adds semantic / paraphrase search. Lexical search is ready as soon as setup.sh exits; vector/hybrid search becomes available when the background build finishes. |

Override: `./setup.sh --indexes none|bm25|hybrid`

**Background build progress** for the hybrid level:
- `tail -f .vector-build.log` from the workspace root, or
- call the MCP `index_status` tool — it reports `"building": true` with the
  process PID and the last log lines while the embed job is running.

`web` mode forces `indexes=none` (there are no local clones to index).

**Determining the active state:** Read `.setup-stamp.json` in the skill workspace
and look at `"mode"` and `"indexes"`. If the file does not exist, run `./setup.sh`.

## Goal

Depending on the active mode, the skill provides access to these resources either
locally (via cloned repositories) or remotely (via web APIs):

1. The **Slicer source code** – the official C++/Python repositories that make up the
   application.  *(Local in full and lightweight modes.)*
2. The **Extensions Index** – a machine‑readable list of third‑party extensions and their
   repositories.  The skill should iterate through the index files and clone each listed
   repository so that extension code is available for searching.
3. The **Discourse archive** – a mirror of the Slicer Discourse forum content (see
   https://github.com/pieper/slicer-discourse-archive) to allow question‑answering based on
   past community discussions.
4. The **NA-MIC Project Week repository** – a collection of tutorials, presentations, and code
   from NA-MIC Project Weeks (https://github.com/NA-MIC/ProjectWeek), providing additional
   examples and educational materials for Slicer programming. The `scripts/extract_pw_stats.py`
   utility can be used to extract statistics about project weeks, including participant lists
   and project counts. This is useful for finding papers, collaborations, or expertise by
   searching for people who have attended NA-MIC Project Weeks.
5. **Build dependencies** (VTK, ITK, CTK, etc.) – cloned from SuperBuild references.
   *(Local in full mode only.  In other modes, browse via GitHub API.)*
6. **Coding conversations** (optional) – JSONL transcripts of AI-assisted coding sessions
   captured by the [CodingChats](https://github.com/pieper/CodingChats) VS Code extension.
   *(Cloned in full and lightweight modes when the repo exists.)*

With these resources available locally, the agent can use standard command‑line tools
(`git grep`, `grep`, `find`, etc.) to search for symbols, examples, documentation,
Python modules, build configurations, and other snippets that help it craft accurate and
precise responses to programming questions about Slicer.

> 📁 Repositories are checked out into subdirectories of the skill workspace named
> `slicer-source`, `slicer-extensions`, `slicer-discourse`, `slicer-dependencies`,
> `slicer-projectweek`, and `coding-chats` respectively.  You are free to override these paths by setting the
> `SLICER_SRC_DIR`, `SLICER_EXT_DIR`, `SLICER_DISCOURSE_DIR`, `SLICER_DEP_DIR`,
> `SLICER_PROJECTWEEK_DIR`, and `CODING_CHATS_DIR` environment variables before running the setup script.

---

## Prerequisites


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pieper/slicer-skill](https://github.com/pieper/slicer-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
