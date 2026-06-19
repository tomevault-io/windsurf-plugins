---
trigger: always_on
description: >
---


# CV Building Pipeline

End-to-end CV creation: from any input format → semantic accumulation → normalized master doc
→ quality-gated CV YAML → professional PDFs → strategic interview materials.

**Core Philosophy: Premix Storage & Molds**

Think of the master doc as **premix storage** — a neutral, ever-growing repository of your entire career.
Think of each CV as a **mold** — a specific shape poured from the premix for a particular target.

- **Premix (master.md)** — infinite potential, neutral, always accumulating. No tailoring lives here.
- **Molds (cv.yaml)** — specific tailoring pulled from premix. Quality-gated, industry-appropriate, role-targeted.
- **Zero-assumption input** — accepts anything: full biography, draft CV, fractals, LinkedIn paste, conversation. No format required.
- **Sidecar changelog** — all additions/changes/removals tracked in `cv_files/CHANGELOG.md`. No VCS assumed.

**Entry Points:**
- `/cv-building` or "build my CV" → Full pipeline (accumulate → normalize → quality gate → render → package)
- "just want to add to my master" or "just want to fill/populate the master" → Accumulation only (add to master, update changelog)
- "remove X from my master" or "take out this role" → Content removal (delete from master, update changelog)
- "just render my CV" → Quality gate + render only (requires existing master)
- "prepare interview materials" → Strategic package generation only

**Prerequisites:**
- `rendercv` Python package: `uv tool install "rendercv[full]"` (or `pip install "rendercv[full]"`)
- `typst` compiler (installed with rendercv[full])

---

## Pipeline Overview

```
Any Input → Semantic Accumulation → Master Normalization → Quality Gate (cv.yaml) → Package
     ↓              ↓                      ↓                      ↓                      ↓
Free-form     Merge with existing     Organized, neutral     Tailored, criteria-    PDFs + strategic
CV/fractals/  master doc              master doc             matched YAML           materials
conversation
```

---

## Step 1: Semantic Accumulation

Accept any input and merge it into the growing knowledge base.

### First-Time Setup

If `cv_files/master.md` doesn't exist:
1. Create the `cv_files/` directory: `mkdir -p cv_files`
2. Initialize `cv_files/master.md` from the template: [references/master/master.template.md](references/master/master.template.md)
3. Create `cv_files/CHANGELOG.md` with a header and first entry
4. Proceed with accumulation as normal

### Multi-Persona (Optional)

If the user needs separate identities (stage name, anon web3 identity, pen name),
use suffixed master docs: `cv_files/master_<persona>.md`.

- `cv_files/master_stage-name.md` — Actor's public identity
- `cv_files/master_anon.md` — Anon web3/DAO resume
- `cv_files/master_real-name.md` — Personal identity

Each persona gets its own master and changelog. Cross-persona accumulation is never assumed.
If in doubt, default to single `master.md`.

### Input Types (No Assumptions)

| Input | Example | How to Handle |
|-------|---------|---------------|
| **Full biography** | "Here's my complete career history..." | Parse all sections into master |
| **Existing CV** | User pastes or uploads a resume | Extract all data points |
| **Draft tailored CV** | User has a CV for a specific role | Extract raw facts, strip tailoring |
| **Fragments** | "I also led a team of 5 at Company X" | Add to existing role or create new |
| **LinkedIn export** | JSON or text from LinkedIn | Parse structured data |
| **Conversation** | User describes achievements in chat | Extract and structure in real-time |
| **Correction** | "Actually my end date was March, not February" | Confirm, update existing, log in changelog |

### Accumulation Rules

1. **Never discard** — all user-provided data stays in master unless explicitly asked to remove
2. **Always merge** — new input supplements existing master, doesn't replace it
3. **Track everything** — every addition/change/removal logged in changelog
4. **No completeness assumption** — master can be partial, sparse, or empty. Always ready for more.
5. **Explicit removal only** — if user says "remove X" or "take out this role", delete from master and log in changelog. Never remove anything without explicit instruction.
6. **Confirm corrections** — when the user corrects existing info, acknowledge the change before applying: *"Understood, updating [field] from [old value] to [new value]."* Replace the value entirely (master is current truth, not a history doc), and log the change in the changelog.
7. **Detect and merge recurring details** — when new input overlaps with existing entries (same company, same role, overlapping dates), enrich the existing entry instead of creating a duplicate. Confirm before merging: *"This looks like it overlaps with your [Role] at [Company]. Should I add these details to the existing entry, or is this a different position?"* This prevents accidental double-entry from typos or fragmented input (merge conflicts).

### Probing Questions

When input is vague or brief, use targeted questions to excavate career details.
**Question bank:** [references/master/question-bank.md](references/master/question-bank.md)

Pick the most relevant questions per role rather than asking all of them. Focus on:
- **Context:** What would break if you disappeared?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fakhriaunur/cv-building](https://github.com/fakhriaunur/cv-building) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
