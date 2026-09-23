---
trigger: always_on
description: Built and used by [santifer](https://santifer.io) to evaluate 740+ offers, generate 100+ tailored CVs, and land a Head of Applied AI role. The archetypes, scoring, and negotiation scripts reflect that search; his portfolio is also open source: [cv-santiago](https://github.com/santifer/cv-santiago).
---

# Career-Ops -- AI Job Search Pipeline

## Origin

Built and used by [santifer](https://santifer.io) to evaluate 740+ offers, generate 100+ tailored CVs, and land a Head of Applied AI role. The archetypes, scoring, and negotiation scripts reflect that search; his portfolio is also open source: [cv-santiago](https://github.com/santifer/cv-santiago).

**It works out of the box, but it's designed to be made yours.** You (AI Agent) can edit the user's files: they say "change the archetypes to data engineering roles" and you do it. That's the whole point.

## Data Contract (CRITICAL)

Two layers — full list in `DATA_CONTRACT.md`:

- **User Layer (NEVER auto-updated; personalization goes HERE):** `cv.md`, `config/profile.yml`, `modes/_profile.md`, `modes/_custom.md`, `article-digest.md`, `portals.yml`, `data/*`, `documents/*`, `reports/*`, `output/*`, `interview-prep/*`
- **System Layer (auto-updatable; DON'T put user data here):** `modes/_shared.md` and all other modes, `AGENTS.md`, `CLAUDE.md`, `CODEX.md`, `OPENCODE.md`, `KIMI.md`, `GEMINI.md`, `*.mjs` scripts, `dashboard/*`, `templates/*`, `batch/*`

**THE RULE: When the user asks to customize facts or targeting (archetypes, narrative, negotiation scripts, proof points, location policy, comp targets), ALWAYS write to `modes/_profile.md` or `config/profile.yml`. When they ask for procedural house rules, custom workflows, output preferences, or automations, write to `modes/_custom.md` (copy it from `modes/_custom.template.md` if missing). NEVER edit `modes/_shared.md` for user-specific content.** This ensures system updates don't overwrite their customizations.

**Path Resolution Override & Precedence:**
The User Layer location (Data Root) is resolved dynamically using the following precedence order:
1. **Environment Variables:** `CAREER_OPS_ROOT` or `CAREER_OPS_DATA_DIR` overrides the root path (resolved relative to the repository root if it is a relative path).
2. **Marker File:** If no environment variable is set, a `.career-ops-data` file in the repository root containing an absolute or relative path to the user data directory is used.
3. **Repository Default:** If neither is present, the repository root directory itself is used.

**Tracker Path & Canonical Writes:**
- **Explicit override:** `CAREER_OPS_TRACKER` environment variable overrides the applications tracker file path. Relative paths are resolved relative to the repository root directory.
- **Reading:** If no override is set, reading resolves to `{DATA_ROOT}/data/applications.md` if it exists; otherwise falls back to `{DATA_ROOT}/applications.md`.
- **Writing:** All write operations (first-run creation or merge operations) target the canonical location `{DATA_ROOT}/data/applications.md` (or the explicit `CAREER_OPS_TRACKER` override).

## Source-of-Truth Boundary (CRITICAL)

User-facing content (CV, cover letters, application emails, form answers, recruiter outreach) is generated **exclusively** from these files plus statements the user makes directly in the current conversation. The list is tiered by trust level (#2947) — read both tiers before generating content, but treat them differently for quantified claims:

**Primary / user-authored (full trust — the ground truth for facts):**

- `cv.md` · `article-digest.md` · `config/profile.yml` · `modes/_profile.md` · `writing-samples/`
- `modes/_custom.md` (procedural/style rules only — never introduces factual claims)
- `voice-dna.md` (voice/style only — never introduces factual claims)

**Derived / accumulated (narrative + phrasing trust; NOT automatically cv.md-equivalent for numbers):**

- `interview-prep/story-bank.md` and `interview-prep/{company}-{role}.md` (the user's own STAR stories and prep notes; consumed by `interview` and `apply`/`match-star`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [career-ops-hq/career-ops](https://github.com/career-ops-hq/career-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
