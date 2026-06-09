---
trigger: always_on
description: This repository is a local Codex-managed toolkit for:
---

# AGENTS.md — Zotero Research Automation for Codex

## Purpose

This repository is a local Codex-managed toolkit for:

1. Searching scholarly papers by topic.
2. Using safe public academic metadata APIs instead of scraping Google Scholar or publisher websites.
3. Deduplicating and ranking papers.
4. Importing selected references into Zotero through the Zotero Web API.
5. Creating a reusable Codex Skill at `skills/zotero_research/SKILL.md`.

The expected user workflow is:

```bash
python research_to_zotero.py --topic "retrieval augmented generation evaluation" --limit 20 --since 2020 --dry-run
python research_to_zotero.py --topic "retrieval augmented generation evaluation" --limit 20 --since 2020 --collection "RAG Evaluation"
```

Always prefer a dry run before writing to Zotero.

---

## Repository Context

The user already has:

- Ubuntu installed.
- Zotero desktop installed on Ubuntu.
- Zotero browser connector installed.
- A Zotero account with sync enabled or planned.
- A Zotero Web API key with write access.
- Existing project files:
  - `README.md`
  - `requirements.txt`
  - `zotero_add.py`

Codex should extend this project rather than replacing it blindly.

---

## Non-Negotiable Safety Rules

1. Never print, log, commit, or expose the Zotero API key.
2. Never hard-code API keys in source files.
3. Use environment variables and optionally `.env`.
4. Add `.env` to `.gitignore`.
5. Do not scrape Google Scholar.
6. Do not bypass publisher paywalls.
7. Do not download paywalled PDFs.
8. Do not mass-import hundreds of papers without a dry run.
9. Always deduplicate before importing.
10. If metadata quality is poor or uncertain, show the paper in dry-run output but do not auto-import it unless the user explicitly requests it.

---

## Target Architecture

```text
User topic
  ↓
research_to_zotero.py
  ↓
search_papers.py
  ↓
OpenAlex / Crossref / arXiv / Semantic Scholar
  ↓
Normalized paper objects
  ↓
Deduplication and ranking
  ↓
zotero_add.py
  ↓
Zotero Web API
  ↓
Zotero desktop sync
```

The implementation should use public scholarly metadata APIs:

- OpenAlex: default broad scholarly search.
- Crossref: DOI metadata enrichment.
- arXiv: preprints for CS, math, physics, statistics, quantitative biology, quantitative finance, and electrical engineering.
- Semantic Scholar: optional, useful for citation count and relevance metadata.

---

## Required Files

Codex should create or update the following files:

```text
README.md
requirements.txt
.env.example
.gitignore
search_papers.py
zotero_add.py
research_to_zotero.py
skills/zotero_research/SKILL.md
```

Optional but recommended:

```text
tests/
tests/test_normalize.py
tests/test_deduplicate.py
```

---

## Python Dependencies

`requirements.txt` should include at least:

```txt
pyzotero
requests
python-dotenv
feedparser
rapidfuzz
rich
```

Optional test dependency:

```txt
pytest
```

Purpose of dependencies:

- `pyzotero`: interact with Zotero Web API.
- `requests`: call academic metadata APIs.
- `python-dotenv`: load local `.env`.
- `feedparser`: parse arXiv Atom API responses.
- `rapidfuzz`: fuzzy title deduplication.
- `rich`: clear terminal output.

---

## Environment Variables

Create `.env.example`:

```env
ZOTERO_USER_ID=your_zotero_user_id
ZOTERO_API_KEY=your_zotero_api_key
ZOTERO_LIBRARY_TYPE=user
ZOTERO_COLLECTION_NAME=Auto Imported Papers

# Optional
SEMANTIC_SCHOLAR_API_KEY=
OPENALEX_MAILTO=your_email@example.com
```

`ZOTERO_LIBRARY_TYPE` should usually be `user`.

For group libraries, allow the user to set:

```env
ZOTERO_LIBRARY_TYPE=group
ZOTERO_GROUP_ID=your_group_id
```

but group support can be secondary.

---

## `.gitignore` Requirements

Ensure `.gitignore` contains:

```gitignore
.env
.venv/
__pycache__/
*.pyc
.pytest_cache/
```

---

## Common Paper Schema

All search sources must be normalized into this schema:

```python
{
    "title": str,
    "authors": [
        {"firstName": str, "lastName": str}
    ],
    "year": str | None,
    "date": str | None,
    "doi": str | None,
    "url": str | None,
    "abstract": str | None,
    "publication_title": str | None,
    "source": str,
    "source_id": str | None,
    "arxiv_id": str | None,
    "openalex_id": str | None,
    "semantic_scholar_id": str | None,
    "citation_count": int | None,
    "item_type": "journalArticle" | "preprint" | "conferencePaper" | "bookSection" | "book"
}
```

Rules:

- DOI should be normalized to lowercase and stripped of URL prefixes.
- Titles should be stripped and normalized for deduplication.
- Authors should use Zotero creator dictionaries.
- If first/last name parsing is uncertain, put the full name in `lastName` and leave `firstName` empty.
- Prefer `journalArticle` unless there is a clear reason to use another Zotero item type.
- arXiv papers can be stored as `preprint` if the Zotero API template supports it; otherwise use `journalArticle` with arXiv metadata in `extra`.

---

## `search_papers.py` Requirements

Implement these functions:

```python
def search_openalex(topic: str, limit: int = 20, since: int | None = None) -> list[dict]:
    ...

def search_crossref(topic: str, limit: int = 20, since: int | None = None) -> list[dict]:
    ...

def search_arxiv(topic: str, limit: int = 20, since: int | None = None) -> list[dict]:
    ...


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zjjues/zotero-skill](https://github.com/zjjues/zotero-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
