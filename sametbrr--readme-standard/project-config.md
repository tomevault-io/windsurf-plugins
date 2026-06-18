---
trigger: always_on
description: |
---


## Standard structure

### README.md (English)

```
[badges]
[nav line]            ← include only if ≥8 sections or ≥200 lines (Rule 14)

# Project Name

One-line description.

> 🇹🇷 Türkçe için [README.tr.md](README.tr.md)

[value paragraph]     ← include only if published package/product (Rule 20)

---

## Quick Start
## Features           ← table form if ≥7 features (Rule 19)
## Requirements
## Installation
   ### Uninstall      ← include only if project installs something persistent
## Usage              ← per-command template if CLI/plugin (Rule 21)
## Configuration      ← include only if user-facing config exists
## How It Works       ← include only if internal mechanics are non-obvious
## Project Structure  ← include only for multi-component projects
## Limitations        ← include only if known user-facing constraints exist
## Troubleshooting    ← include only if predictable failure scenarios exist
## Contributing       ← include only if contributions are accepted
## Community          ← include only if community channels exist
## Documentation      ← include only if external docs site exists
## License
[footer]              ← include only if GitHub repo and ≥200 lines (Rule 23)
```

Section selection is governed by **Conditional sections** below: a conditional
section is included only when its detection signal is present, and is never
emitted empty or as a placeholder.

### README.tr.md (Turkish)

Mirror of README.md with these rules:
- First content line (after badges): `> 🇬🇧 For English see [README.md](README.md)`
- All prose and section headings: translated — see [references/tr-translations.md](references/tr-translations.md)
- Badges: identical (same shields.io URLs)
- Code blocks: never translated

## Conditional sections

Each optional section has a defined inclusion condition and detection signal.
Check the signals during Create and Fix modes; include the section only when
the signal is present.

| Section | Placement | Include when | Detection signal |
|---|---|---|---|
| Value paragraph | After TR link, before first `---` | Published package/product | npm/NuGet/PyPI badge generated; `package.json` not `private` |
| `### Uninstall` | Subsection of Installation | Project installs something persistent | global install (`-g`), installer script, hook/MCP config writes, PATH changes |
| Configuration | After Usage | User-facing config exists | config file schema, env vars, settings file in code |
| How It Works | After Configuration | Internal mechanics non-obvious | multi-step pipeline, background process, index/cache mechanism |
| Project Structure | After How It Works | Multi-component project | multiple top-level modules/packages |
| Limitations | After Project Structure | Known user-facing constraint exists | documented constraint, known issue, platform limit |
| Troubleshooting | After Limitations | Predictable failure scenarios exist | external dependency (gh, docker, API key), auth/network step, multi-step install |
| Contributing | After Troubleshooting | Contributions accepted | `CONTRIBUTING.md`, public repo + dev/test scripts |
| Community | After Contributing | Community channels exist | Discord/Slack/forum/discussions link |
| Documentation | After Community | External docs site exists | docs URL (homepage field, published docs/) |
| Footer | After License, last element | GitHub repo + README ≥200 lines | git remote is GitHub + line count |

Rules:

- **Placement is relative order, not a dependency.** The "Placement" column
  defines the canonical order of the table; "After X" does not require X to
  exist. When a predecessor is omitted, the section slots in after the nearest
  *present* section that precedes it in the table (e.g. if How It Works is
  omitted, Project Structure goes directly after Configuration — or after
  Usage if Configuration is also omitted).
- **User instruction always overrides detection** — "uninstall ekleme" / "skip
  troubleshooting" removes the section even when the signal is present;
  "troubleshooting ekle" adds it even without a signal (ask the user for
  content if none is detectable).
- A conditional section whose condition is not met is **omitted entirely** —
  never emitted empty, as "N/A", or as a placeholder (that is a FAIL, Rule 17).
- In Audit mode, conditional sections produce **⚠️ WARN only, never ❌ FAIL**:
  - section present but signal absent → `⚠️ 'Uninstall' present but project installs nothing persistent`
  - signal present but section missing → `⚠️ Project installs globally; 'Uninstall' section recommended`
- Section names and order are fixed (Rule 11 discipline applies to conditional
  sections too). README.tr.md must contain exactly the same section set.

## Mode workflows

### Audit mode

1. Read README.md and README.tr.md (if exists)
2. Check each rule against [references/rules.md](references/rules.md)
3. Check conditional sections two-way (section↔signal, WARN only — see **Conditional sections**)
4. Print checklist — ✅ PASS / ❌ FAIL / ⚠️ WARN per rule, with line reference where it fails
5. Do NOT write any files

### Create mode

1. Detect project type and read metadata → see **Project type detection** below
2. Evaluate conditional section signals → see **Conditional sections** (user instructions override)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sametbrr/readme-standard](https://github.com/sametbrr/readme-standard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
