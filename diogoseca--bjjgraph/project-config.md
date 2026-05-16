---
trigger: always_on
description: BJJ knowledge graph and state machine as a static site. **Site:** https://bjjgraph.org | **Repo:** https://github.com/diogoseca/bjjgraph
---

# BJJGraph - AI Development Guide

BJJ knowledge graph and state machine as a static site. **Site:** https://bjjgraph.org | **Repo:** https://github.com/diogoseca/bjjgraph

---

## 1. AI WORKFLOW (MANDATORY)

### Step 1: Read Documentation (In Order)

```
1. CLAUDE.md          ← You are here
2. docs/Architecture.md   ← JSON pipeline, Position model
3. docs/Content.md        ← Standards, validation rules
4. docs/SEO.md            ← Schema markup, keywords, analytics
```

### Step 2: Ask Critical Questions

Before starting any task, clarify:

| Question | Why It Matters |
|----------|----------------|
| **Scope** | What files/systems does this touch? |
| **Context** | What existing patterns should I follow? |
| **Constraints** | What must NOT change? |
| **Success** | How will we verify this worked? |

### Step 3: Deploy Subagents

Match task to engineering personality, then deploy parallel or sequential:

| Personality | Best For | Example Tasks |
|-------------|----------|---------------|
| **Systematic Migrator** | Repetitive, pattern-following | Bulk content updates, format changes |
| **Complexity Handler** | Multi-step logic, debugging | Validation errors, template fixes |
| **Infrastructure Specialist** | CI/CD, deployment, performance | GitHub Actions, build optimization |
| **Ruthless Editor** | Documentation, DRY, deletion | Consolidate docs, remove dead code |
| **Paranoid Tester** | Edge cases, validation | Schema validation, error handling |
| **Architect** | System design, refactoring | New features, structural changes |

**Parallel:** Independent tasks (audit multiple files, run multiple checks)
**Sequential:** Dependent tasks (validate → fix → regenerate → build)

### Step 4: Update Documentation

Follow **C-UD pattern** after completing work:
- **Create** new docs only if essential (prefer updating existing)
- **Update** existing docs with new learnings
- **Delete** outdated information immediately

---

## 2. FORBIDDEN ACTIONS

### Never Do These

| Action | Why | Do Instead |
|--------|-----|------------|
| Edit generated `.md` in `content/` | Overwrites on regeneration | Edit `.json` source in `content/` (data) or templates in `templates/` (schemas/Jinja2) |
| Skip validation before commits | Breaks build, bad data | Run `npm run regenerate:build` |
| Create files >1000 lines | Unmaintainable | Split into focused modules |
| Add docs without updating index | Orphaned content | Update relevant README/index |
| Commit secrets or API keys | Security breach | Use `.env` files, check `.gitignore` |
| Guess at wikilink targets | Broken links | Verify file exists first |
| Add emojis to content files | Inconsistent styling | Only use in docs if necessary |

### File Size Limits

- **Code files:** <500 lines preferred, <1000 max
- **Documentation:** <300 lines preferred
- **JSON source files:** No limit (data files)

---

## 3. PROJECT STRUCTURE

```
bjjgraph/
├── CLAUDE.md                    # AI workflow (this file)
├── README.md                    # Quick start for contributors
├── graph.json                   # Generated graph data for visualization
├── content/                     # JSON source + generated markdown
│   ├── Positions/               # 85+ positions (*.json = SOURCE, *.md = GENERATED)
│   ├── Transitions/             # 1000+ transitions (*.json = SOURCE, *.md = GENERATED)
│   ├── Submissions/             # 150+ submissions (*.json = SOURCE, *.md = GENERATED)
│   ├── Systems/                 # Expert systems
│   └── Principles/              # Fundamental principles
├── templates/                   # JSON schemas + Jinja2 templates
│   ├── Positions/               # TEMPLATE-*.json schemas + *.md.jinja2 templates
│   ├── Transitions/             # TEMPLATE-*.json schemas + *.md.jinja2 templates
│   ├── Submissions/             # TEMPLATE-*.json schemas + *.md.jinja2 templates
│   ├── Principles.json          # Principles data
│   ├── Systems.json             # Systems data
│   ├── Transitions.json         # Hub page transition data
│   ├── Submissions.json         # Hub page submission data
│   └── votes.json               # Community voting data
├── docs/
│   ├── Architecture.md          # JSON pipeline, Position model
│   ├── Content.md               # Standards, validation rules
│   └── SEO.md                   # Schema markup, keywords, analytics
├── scripts/
│   ├── validate_json.py         # JSON schema validation
│   ├── validate_graph_integrity.py  # Graph consistency checks
│   ├── regenerate_content_json.py   # Auto-fill TODOs in JSON
│   ├── regenerate_md_from_json.py   # Regenerate markdown from JSON
│   ├── regenerate_category_hub_pages.py  # Generate category hub pages
│   ├── regenerate_graph.py      # Generate graph.json
│   ├── regenerate_votes.py      # Generate voting data
│   ├── explode_graph_connections.py  # Expand graph connections
│   ├── regenerate_list_of_content_files_to_fix.sh  # List files needing fixes
│   └── select_oldest_files.sh   # File selection for bot
├── branding/                    # Brand assets and icons
├── source/                      # Quartz code only (MIT)
│   ├── quartz/                  # Static site generator components
│   ├── quartz.config.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diogoseca/bjjgraph](https://github.com/diogoseca/bjjgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
