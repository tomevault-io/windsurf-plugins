---
trigger: always_on
description: Docs & project conventions — structure, locations, format, version control
---


# Docs & Project Conventions

> Workspace-level rules for documentation, rambles, and file organization.
> Applies when creating/editing .md files or project docs.

---

## Dev Structure

**Multi-project workspace.** Dev folder holds many projects; structure scales for each.

```
c:\dev\
├── projects\           ← Project code (one folder per project)
├── ai\                 ← AI-related
│   ├── cursor\         ← Cursor docs, rules
│   ├── chats\          ← AI chat exports
│   └── projects\       ← AI-specific notes per project
├── docs\               ← Human docs, rambles
└── .zips\              ← Archives, versions, pre-change, version-archives (per project)
```

**AI vs human:** AI does most of the code; human directs, reviews, and contributes. See [DEV_OVERVIEW.md](../docs/DEV_OVERVIEW.md).

---

## Rules Maintenance

**When the user instructs to add a rule, make something a rule, or update the rules** (e.g. "add a rule for that", "make that a rule", "update the rules so that..."):

1. **Update** `ai/cursor/rules/docs-conventions.mdc` with the new or changed rule.
2. **Do it** — do not just acknowledge; actually edit the file.
3. **Log** the change in ACTIVITY_LOG.md (timestamp, what, why, how).

The .cursor/rules version is a hard link to this file, so it updates automatically.

---

## Doc Locations

| Location | Purpose |
|----------|---------|
| `c:\dev\ai\cursor\docs\` | AI-created/edited docs — overviews, plans, TODOs, summaries |
| `c:\dev\docs\` | Human-authored docs, rambles |
| `c:\dev\ai\chats\` | AI chat exports |
| `c:\dev\ai\projects\` | AI-specific notes per project |

**Single docs location for AI:** All .md files created or edited by AI go in `c:\dev\ai\cursor\docs\`. Do not create a second docs folder.

---

## Cursor-Generated .md Timestamp

- **Every .md file** created or edited by Cursor must include a timestamp in the header:
  - Format: `**Generated:** YYYY-MM-DD HH:mm` (for new files)
  - Format: `**Last updated:** YYYY-MM-DD HH:mm` (for edits)
- Place at top of file, after title if present

---

## Doc Format (Summaries)

Every .md file should be as accurate, detailed, and comprehensive as possible. Include four summary levels:

| Summary | Purpose |
|---------|---------|
| **Short** | One line or brief — quick scan |
| **Long** | Paragraph or two — full context |
| **Dumb** | Plain language, no jargon — anyone can understand |
| **Technical** | Paths, conventions, details — for implementers |

### Summary Placement

- **Top of file:** Add links to summaries after the title/metadata:
  ```
  [Short](#short) · [Long](#long) · [Dumb](#dumb) · [Technical](#technical)
  ```
- **Bottom of file:** Place the four summary sections (`## Short`, `## Long`, `## Dumb`, `## Technical`) at the end of the document.
- Main content goes in the middle.

### Edit Fields (user input areas)

Markdown has no native form fields. Use **placeholder format** for areas where user input goes:

```
> [Edit: hint or example — replace this with your content]
```

- **Format:** `[Edit: hint]` — user replaces the bracketed text with their input
- **Use in:** PROJECT_SPEC.md, template question files, any .md where user fills in details
- **Example:** `> [Edit: your-project-name — lowercase, hyphenated]`

---

## Project Doc Structure

**Adding a new project:** Use `/dev-new-project` (prompts for name, details, template) or manually create folders in `projects/<project-name>/`, `ai/cursor/docs/<project-name>/`, `docs/rambles/<project-name>/`, `.zips/<project-name>/`. Add to README.md and OVERVIEW.md Projects table.

- **Project code** lives under `c:\dev\projects\<project-name>/` (one folder per project)
- **Project-specific docs** live under `c:\dev\ai\cursor\docs\<project-name>/` (lowercase, hyphenated folder name)
- **Features/functionality subfolder:** Each project has `ai/cursor/docs/<project>/features/` for what/why/how docs
- **3+ .mds rule:** When a project docs folder has 3 or more .md files, create a subfolder to organize them
- **Each project** is allowed **one README.md** in its root that links to needed files
- README.md should link to `c:\dev\ai\cursor\docs\<project>/OVERVIEW.md`

---

## Rambles

- **Location:** `c:\dev\docs\rambles\` — rambles live in docs
- **Structure:** `docs/rambles/<project-name>/` (one folder per project)
- Raw notes, ideas, and brain dumps go in the project's rambles folder

---

## .zips

- **All .zip files** live in `c:\dev\.zips\` unless needed inside a project (e.g. build artifact, deployment package)
- **Structure per project:**
  ```
  .zips/<project-name>/
  ├── archives/        — .zip files (not yet extracted)
  ├── extracted/       — history of .zip files that have been extracted
  ├── versions/        — prod version zips (v1.0.zip, v1.1.zip) — only when project is in prod
  ├── pre-change/      — archives + logs before changes to deployed prod
  └── version-archives/ — superseded version zips (see conditions below)
  ```
- **misc/** — Workspace-level zips (not tied to a project): `misc/archives/` and `misc/extracted/`. dev-zip and dev-zip-full outputs go here.
- When extracting: move the .zip from `archives/` to `extracted/` (or add a note) to track history

### Commands (`.cursor/commands/`)

| Command | Purpose |
|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anoncano) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
