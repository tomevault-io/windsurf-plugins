---
trigger: always_on
description: Generate a codetree (comprehensive codebase map as nested markdown) and a codemap.json (visual branch terrain map) for any project. Use when the user asks to map a codebase, generate a codemap, create a codetree, understand a project's branch landscape, or produce a visual overview of a repository's architecture and work-in-progress.
---


# Codemap Skill

Produce two artifacts for a repository:

1. **`<project>-codetree.md`** — a deep nested-list document covering every file, system, branch, and possible task in the codebase
2. **`<project>-codemap.json`** — a terrain map JSON consumed by the Codemap viewer, placing branches as landmarks across a procedurally-generated world

Both files are written to the project's root directory.

---

## Phase 1 — Explore the Codebase

Explore thoroughly before writing anything. The codetree's quality depends entirely on how well the codebase is understood.

### What to gather

**Repository structure**
```bash
ls <project-root>/
# Read README.md, package.json / pyproject.toml / Cargo.toml / go.mod
# Read the main config files: vite.config.*, next.config.*, vercel.json, etc.
```

**Source files** — read every meaningful file in `src/`, `lib/`, `app/`, `api/`, `backend/`, etc. For large files (>500 lines), read enough to understand:
- What the file does
- Its key exports, functions, or components
- Dependencies on other files
- Architectural patterns it establishes

**Git branches and history**
```bash
git -C <project-root> branch -a
git -C <project-root> remote get-url origin
git -C <project-root> symbolic-ref refs/remotes/origin/HEAD
git -C <project-root> log --oneline --all --graph -80

# Full dated commit log for timeline reconstruction
git -C <project-root> log --format="%h %ad %s" --date=short -120

# If the log is long, page through it
git -C <project-root> log --format="%h %ad %s" --date=short -120 --skip=120

# PR merges only (useful for epoch boundaries)
git -C <project-root> log --oneline --merges
```

**What to extract from the history:**
- First commit date and message (the "epoch 0" anchor)
- HEAD commit hash, date, and message
- Origin remote URL and default branch, if present. If there is no remote, omit `repo`.
- Total commit count on main: `git rev-list --count HEAD`
- List of merge commits (identifies PRs and branch landings)
- Any large direct-to-main bursts (multiple commits same day, no merge parent)
- Gaps between activity clusters (quiet periods between epochs)
- Author names from commit log

**Related projects** — if sibling directories look related (e.g. `project-ios`, `project-map`), do a quick `ls` and README read.

### What to capture per file
- Purpose in one sentence
- Key functions / components / exports
- Notable patterns or technical decisions
- Dependencies on other files

---

## Phase 2 — Write the Codetree

Write `<project>-codetree.md` in the project root. The codetree is a comprehensive nested markdown document. Use `##` for top-level sections and nested `-` lists for everything else.

### Required sections

#### 1. Project Overview & Architecture
- What the product does (one paragraph)
- High-level data flow (request → processing → response)
- Interaction modes or user flows
- Repository layout (top-level dirs and what they contain)

#### 2. Frontend — `src/` (or equivalent)
One subsection per significant file or component group. For each:
- What it does
- Key sub-topics a developer might ask about (e.g. "Explain the generation state machine")
- Refactor targets if the file is large (note with "**Refactor targets (see §6):**")
- Delete candidates if stale code exists (note with "**Delete candidates (see §7):**")

#### 3. Backend / API — `api/` or `backend/` (or equivalent)
Same treatment as Frontend. Break into subsections per route file, handler, or service. For large library files, enumerate their major sections:
- Configuration
- Data models / error types
- Core logic functions (list each with a one-line description)
- Utilities

#### 4. Configuration & Deployment
- Build tooling (Vite, webpack, etc.) — multi-entry points, output config
- Deployment config (Vercel, Netlify, Dockerfile, etc.)
- Environment variables — group by purpose (model keys, infra, runtime, feature flags)

#### 5. Active Branches & Directions of Work
One subsection per branch (or logical group of related branches). For each:
- Branch name(s)
- What the work is (description from recent commit messages)
- Direction — what it's trying to achieve
- Status — in progress / merged / stale / experimental

Group related branches together: e.g. all `username/scaling-*` variants under one heading.

#### 6. Refactoring Opportunities
Specific, actionable extraction targets. Name the current file, the proposed new file/module, and what moves there. Format:
```
### 6.1 `src/App.jsx` decomposition (~N lines)
- **Extract WaitRoom** → `src/components/WaitRoom.jsx`
- **Extract generation SSE logic** → `src/lib/useGeneration.js`
```

#### 7. Deletion Candidates
Things that appear safe to remove: legacy keys, dead endpoints, unused model paths, stale branches. Note why each is safe to delete.

#### 8. Features That Could Be Requested
Group by horizon:
- **In-progress / near-term** — features visible in branches, almost ready
- **Product surface expansions** — natural next steps from existing patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarzain/codemap](https://github.com/tarzain/codemap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
