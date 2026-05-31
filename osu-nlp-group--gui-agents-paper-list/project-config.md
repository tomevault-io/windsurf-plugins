---
trigger: always_on
description: This repository is a curated list of research papers on direct GUI Agent research. It tracks ~600+ papers with structured metadata across web, mobile, desktop, and general-GUI settings.
---

# CLAUDE.md - GUI Agents Paper List

## Project Overview

This repository is a curated list of research papers on direct GUI Agent research. It tracks ~600+ papers with structured metadata across web, mobile, desktop, and general-GUI settings.

**The source of truth is [`papers.yaml`](papers.yaml)** (and [`adjacent.yaml`](adjacent.yaml) for non-canonical adjacent entries). All other artifacts — `README.md`, the site, the per-axis statistics images — are auto-generated from these YAML files.

## Canonical Update Workflow

This repository is updated locally. GitHub Actions is no longer part of the official workflow.

After editing `papers.yaml`, regenerate every derived artifact locally with:

```bash
bash scripts/update_repo.sh
```

The pipeline runs `scripts/regen.py`, which:
1. Loads `papers.yaml` and `adjacent.yaml`.
2. Sorts entries newest-first, writes the YAML files back canonicalised (sorted, ISO date format).
3. Renders `README.md` directly from `readme_template/template.md` (no intermediate fragment files).
4. Emits `readme_template/statistics/{quarterly_trend,keyword_bar_chart}.png`.

Then review the diff, push, and deploy the site:

```bash
git status --short
git diff --stat
git add papers.yaml adjacent.yaml README.md readme_template/statistics scripts tests requirements.txt pyproject.toml uv.lock .gitignore CLAUDE.md
git commit -m "..."
git push
bash site/scripts/deploy.sh   # always deploy after any papers.yaml or site/ change
```

Dependencies are in `requirements.txt` and `pyproject.toml`. With `uv`:

```bash
uv sync
```

## Repository Structure

```
paper_repo/
├── papers.yaml                  ← canonical source of truth (canonical entries)
├── adjacent.yaml                ← canonical source of truth (adjacent entries)
├── README.md                    ← auto-generated; do not edit
├── CLAUDE.md                    ← this file
├── scripts/
│   ├── update_repo.sh           ← entrypoint; regenerates everything
│   ├── regen.py                 ← single-pass: sort YAML + render README + emit charts
│   └── sync_dates_from_paper_db.py   ← push verified dates from paper_db into papers.yaml
├── readme_template/
│   ├── template.md              ← README template with {{placeholders}}
│   ├── statistics/*.png         ← auto-generated stats charts
│   └── logs/                    ← regen warnings (.gitignored)
├── tests/
│   └── test_local_update_workflow.py
├── site/                        ← Astro static site → GitHub Pages
└── pyproject.toml / requirements.txt / uv.lock
```

## How to Add a Paper

Edit `papers.yaml` and append a YAML object with this schema. Only `title` and `link` are required; everything else is optional but most should be filled in:

```yaml
- title: "Paper title (preserve official capitalization)"
  link: https://arxiv.org/abs/2404.07972      # primary canonical link
  authors:
    - First Author
    - Second Author
  institutions: [OSU, CMU]
  date: "2024-04-11"                            # ISO YYYY-MM-DD, or YYYY-MM, or YYYY
  publisher: "NeurIPS 2024"                     # or "arXiv", "ICLR 2025 (Spotlight)", "TMLR", …
  envs: [Desktop]                               # one or more of: Web, Mobile, Desktop, General GUI
  keywords: [benchmark, OSWorld]                # comma-free list, no [] brackets
  tldr: |
    1–3 sentence factual summary of the paper.
  arxiv_id: "2404.07972"                        # optional; auto-populated if `link` is an arXiv URL
  sources:                                      # optional — extra discoverable links, all keys optional
    arxiv: https://arxiv.org/abs/2404.07972
    openreview: https://openreview.net/forum?id=…
    publisher_page: https://aclanthology.org/…  # ACL Anthology / NeurIPS Proceedings / etc.
    homepage: https://project-page.io/
    code: https://github.com/org/repo
    dataset: https://huggingface.co/datasets/…
  bibtex: |                                     # auto-generated if absent
    @inproceedings{author2024key,
      title = {{…}},
      …
    }
  bibtex_confirmed: false                       # set true after verifying against the official source
```

For `sources:` use what's available. ICLR / NeurIPS post-2017 papers often have OpenReview *as* the publisher's official page — fill `openreview` and leave `publisher_page` empty. ACL/EMNLP/NAACL papers have ACL Anthology as their `publisher_page`.

After editing, run `bash scripts/update_repo.sh`.

### Field Specifications

**Title.** Use the paper's canonical public title from the linked source. Normalize LaTeX or typography artifacts (e.g. `\textsc{...}`, `$^2$`) into readable plain text. Keep official abbreviations (`WALT`, `OS-ATLAS`, `Agent Q`) but don't invent new ones.

**Link.** Derived from `sources:` using this fixed priority: `homepage` → `arxiv` → `openreview` → `publisher_page`. Use the first populated source as `link:`. Only one link goes here — all known links go in `sources:`.

**Authors.** Include all of them.

**Institutions.** Common abbreviations preferred (OSU, MIT, CMU). Use `Unknown` only if no source specifies one.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OSU-NLP-Group/GUI-Agents-Paper-List](https://github.com/OSU-NLP-Group/GUI-Agents-Paper-List) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
