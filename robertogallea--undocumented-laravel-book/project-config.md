---
trigger: always_on
description: > Operational tool for every work session on this book. Keep it concise: it does not
---

# CLAUDE.md - Operating guide for this project

> Operational tool for every work session on this book. Keep it concise: it does not
> duplicate `docs/chapters-overview.md` or `docs/book-index.md`, it points to them as the
> sources of truth.

## 1. Project overview

We are writing an open-source technical book in English titled:

> **Laravel 13: The Unwritten API - A Guide to Usable but Undocumented Concepts**

**Source of truth**: the official source code, tag `v13.22.0` of `laravel/framework`, branch
`13.x` of `laravel/docs`, plus the first-party package repositories at their respective
latest stable versions. Every release of the book states the Laravel version it was verified
against; see `CHANGELOG.md` and the `laravelversion` key in `metadata.yaml`.

**Selection criteria for any entry** (all three required, see `docs/book-index.md` for the
full rationale):

1. Public method/class, realistically usable in an application (not internal plumbing:
   `build*`, `push*`, `setClient`, interface implementations like
   `ArrayAccess`/`JsonSerializable`, etc.).
2. Absent from the corresponding official documentation, verified by exact method/class
   name, not just by general topic.
3. Not a trivial alias with no instructional value of its own, unless explicitly flagged as
   such.

**Mandatory rule**: every entry has at least one real, runnable code example, never
pseudocode (see section 4).

**Structure**: `chapter-0` (Motivation and Methodology, no Part) opens the book; chapters 1-18
are grouped into 8 named Parts; `chapter-19` (Final Chapter - Conclusions, no Part) closes the
main body; `chapter-A` and `chapter-B` are the appendices.

| Part | Title | Chapters |
|---|---|---|
| I | Code Fundamentals | 1-3 |
| II | Eloquent Beyond Basic Relationships | 4-5 |
| III | HTTP, APIs, and Testing | 6-7 |
| IV | Container and Routing | 8-9 |
| V | Authorization, Validation, and Asynchrony | 10-12 |
| VI | Artisan Commands | 13-14 |
| VII | Observing and Communicating | 15-16 |
| VIII | Application Infrastructure | 17-18 |

When a Part is referred to by name in prose, always write `the "Name" Part`, never the bare
proper noun.

For the full per-chapter content (entries, outlines, example overviews) see
`docs/chapters-overview.md`; for audience/format/versioning/length see `docs/book-index.md`.

## 2. Folder structure

```
.
├── CLAUDE.md
├── README.md, CONTRIBUTING.md, LICENSE, LICENSE-CODE, CHANGELOG.md, VERSION
├── build-book.sh, to_pdf.sh, to_epub.sh
├── metadata.yaml, template.tex, epub-template.xhtml, epub.css, section-divider.lua
├── docs/
│   ├── chapters-overview.md, book-index.md, kdp-snapshots.md
│   └── chapters/chapter-N/chapter-N.md + prompts/step-M.md
├── chapters/
│   └── chapter-N/chapter-N-text.md
├── site/                # Docusaurus, synced from chapters/, see section 6
├── .github/workflows/    # ci.yml, deploy-site.yml, build-artifacts.yml
└── code/                 # companion Laravel app, independent git repo, gitignored here
```

Rules (binding):

- **Each chapter is isolated** in its own `chapters/chapter-N/` folder. Do not mix content
  from different chapters in the same file.
- **The spec and prompts for a chapter live in `docs/chapters/chapter-N/`**, not inside
  `chapters/chapter-N/`: the latter holds only the final prose (`chapter-N-text.md`).
- **`docs/` is the planning/spec layer, never the published site's content.** Docusaurus
  reads from `site/docs/`, which is generated and gitignored (see section 6) - never hand-edit
  it, and never confuse it with the top-level `docs/`.
- **The companion application lives in `code/`**, an independently git-tracked, gitignored
  project (see section 6 of `code/README.md`). Every code example in every chapter is
  extracted from there with `git -C code show <tag>:<path>`, never invented.
- **Before drafting any entry, re-verify it against the actual tagged `laravel/framework`
  source and the actual `laravel/docs` branch content** - the specs in `docs/` are a starting
  point written once; the live source is the authority, and may have changed since (a method
  may have been documented, removed, or changed signature).
- **Numbering**: `chapter-0` (motivation/methodology) .. `chapter-18` (last Part chapter),
  `chapter-19` (Final Chapter - Conclusions), then the appendices `chapter-A`, `chapter-B`.
- **`code/` tag convention**: milestone tags `ch01-complete` .. `ch18-complete`, cut when a
  chapter's full example set is done - not a per-increment tag scheme like a single growing
  app, since there is no single continuously-growing example application in this book.

## 3. Reference files and reading order

Before writing or modifying any chapter, always consult, in this order:

1. **`docs/chapters-overview.md`** - per-chapter short description, outline, and example
   overview. Consult it to understand the chapter's purpose and stay coherent with the rest
   of the book.
2. **`docs/book-index.md`** - audience, format, the three-tier example rule, selection
   criteria, Part/chapter list, versioning and length targets.
3. **`docs/chapters/chapter-N/chapter-N.md`** - the chapter's operational spec (Summary /
   Objectives / Requirements / Writing phases).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertogallea/undocumented-laravel-book](https://github.com/robertogallea/undocumented-laravel-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
