---
trigger: always_on
description: provides_properties: [metric-scale, per-pixel-uncertainty, ...]   # nonlocal properties the output carries
---

# Vision Wiki — Schema & Operating Manual

This is a **personal research-synthesis engine** for photogrammetry and
machine-learning. It is not a reading list and not a summarization tool. Its
end-product is **novel pipelines**: each thread maintains the current
best-known end-to-end pipeline(s) for a problem — one per *operating point*
— and every ingested paper is mined for ideas that could improve those
pipelines, either by replacing a stage or by unlocking a *new combination*
of ideas that no single paper has tried. Deep mechanism-level understanding
of each paper, and holistic reasoning about how ideas compose across
papers, are the two activities the wiki exists to support.

You (the LLM) are the sole author and maintainer of the `wiki/` layer. The
human curates sources and asks questions. Never invent citations. A shallow
summary is worse than useless — if you can't explain the mechanism of a new
idea, you don't yet understand the paper.

Domain focus: photogrammetry, SfM, MVS, SLAM, NeRF / 3D Gaussian Splatting,
neural implicit surfaces, feature matching, bundle adjustment, camera
calibration, point-cloud / mesh processing, depth estimation, pose
estimation, and the ML methods that power them (transformers, diffusion,
self-supervised learning, etc.).

---

## 1. Directory layout

```
VisionWiki/
├── CLAUDE.md              # this file — the schema
├── index.md               # content-oriented catalog of every wiki page
├── log.md                 # chronological append-only activity log
├── raw/                   # INBOX — drop anything here, emptied on ingest (§1.1)
├── papers/                # LOCAL CACHE (git-ignored): PDFs, re-downloadable via url: (§1.2)
│   ├── radiance-fields/
│   ├── feature-matching/
│   └── ...
├── articles/              # permanent store: blog posts, web articles, notes (§1.3)
├── assets/                # LOCAL CACHE (git-ignored): images, figures, diagrams (§1.4)
└── wiki/                  # LLM-authored, constantly-maintained knowledge base
    ├── papers/            # one page per ingested paper (the "source summary")
    ├── ideas/             # atomic novel contributions extracted from papers (§1.6)
    ├── stages/            # typed pipeline stages — input/output/invariant schemas (§1.7)
    ├── methods/           # algorithms, architectures, techniques
    ├── concepts/          # general ideas and primitives
    ├── datasets/          # benchmark + training datasets
    ├── people/            # prolific authors / research groups
    ├── threads/           # evolving per-goal pipelines
    └── designs/           # concrete "how to build it" plans
```

### 1.1 Inbox (`raw/`)

`raw/` is a **drop zone / inbox**. The user dumps any files here — PDFs,
markdown clippings, images, screenshots — with whatever messy names they
have. The LLM never reads directly from `raw/` during queries.

On ingest (bare `ingest` with no arguments), the LLM:
1. Scans `raw/` recursively for all files.
2. Classifies each file by type (paper, article, or asset) — §1.5.
3. Renames and moves it to the appropriate permanent store.
4. Deletes the original from `raw/`.

**After a complete ingest, `raw/` should be empty.** A non-empty `raw/`
is the primary signal that work remains.

### 1.2 Paper storage (`papers/`)

`papers/` is a **local cache** of source PDFs / markdown / arXiv HTML.
Git-ignored (PDFs are large + binary). Wiki pages in `wiki/papers/` are
committed; their `url:` field lets any clone re-download on demand.

**Cache behavior**: if the local file at `local_paper:` is missing but
`url:` exists, download it first:
- arXiv: `curl -L -o <local_paper_path> https://arxiv.org/pdf/<id>`
- Other: `curl -L -o <local_paper_path> <url>`

**Naming**: `<FirstAuthor>_<Year>_<Short-Title>.<ext>` (lowercase
kebab-case short title). E.g. `kerbl_2023_3d-gaussian-splatting.pdf`.

**Subfolder taxonomy** — create / merge / restructure as volume demands:

| Subfolder | Scope |
|-----------|-------|
| `radiance-fields/` | NeRF, 3DGS, neural implicit surfaces, novel-view synthesis |
| `feature-matching/` | keypoint detection, descriptor learning, matching |
| `sfm-slam/` | structure from motion, visual SLAM, visual odometry |
| `mvs-depth/` | multi-view stereo, mono/stereo depth |
| `pose-estimation/` | camera / object pose, PnP |
| `mesh-reconstruction/` | surface reconstruction, meshing, point-cloud |
| `fundamentals/` | general ML methods, transformers, diffusion, optimization |
| `datasets-benchmarks/` | dataset papers, benchmark comparisons |

If a paper spans categories, file under its **primary contribution** and
note cross-topic relevance in the wiki page.

### 1.3 Article storage (`articles/`)

Blog posts, tutorials, non-paper text sources. Renamed on ingest to
`<source>_<year>_<short-title>.md`. Same subfolder taxonomy as `papers/`
where applicable.

### 1.4 Asset storage (`assets/`)

Images, figures, diagrams. Renamed to
`<paper-or-article-key>_<descriptor>.<ext>`. Referenced as
`![caption](../assets/<file>)`. Never hot-link external URLs.

### 1.5 Classification rules for ingest

| File type | Destination |
|-----------|-------------|
| PDF (research paper) | `papers/<subfolder>/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdcseacave/VisionWiki](https://github.com/cdcseacave/VisionWiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
