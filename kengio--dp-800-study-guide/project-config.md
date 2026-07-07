---
trigger: always_on
description: Open-source community study guide for the Microsoft **DP-800: Developing AI-Enabled Database Solutions** certification exam. Licensed under MIT (see `LICENSE`). Aligned to the official skills-measured list updated **March 12, 2026**.
---

# CLAUDE.md

Open-source community study guide for the Microsoft **DP-800: Developing AI-Enabled Database Solutions** certification exam. Licensed under MIT (see `LICENSE`). Aligned to the official skills-measured list updated **March 12, 2026**.

The repository is public; contributions arrive as pull requests. Keep this in mind: every change is visible to the community. Prefer additive, well-justified edits over silent rewrites, and verify factual claims against the [official skills measured](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/dp-800) page before merging.

## Repository Structure

```text
dp-800-study-guide/
├── certification/
│   ├── dp-800-overview.md      # Certification index with exam overview and progress tracker
│   ├── 01-database-objects/    # Design and implement database objects
│   ├── 02-programmability-objects/ # Views, functions, stored procedures, triggers
│   ├── 03-advanced-tsql/       # CTEs, window functions, JSON, regex, graph queries
│   ├── 04-ai-assisted-tools/   # GitHub Copilot, MCP, AI security
│   ├── 05-data-security-compliance/ # Encryption, masking, RLS, auditing
│   ├── 06-performance-optimization/ # Configs, isolation levels, query plans
│   ├── 07-cicd-database-projects/   # SQL Database Projects, CI/CD, deployment
│   ├── 08-azure-services-integration/ # DAB, REST/GraphQL, monitoring, CDC
│   ├── 09-models-embeddings/   # External models, embedding generation
│   ├── 10-intelligent-search/  # Full-text, vector, and hybrid search
│   ├── 11-rag/                 # Retrieval-augmented generation
│   └── resources/              # Practice questions, mock exams, exam tips, code examples, appendix, cheat sheets
├── i18n/                       # Community translations — parallel tree per locale, see TRANSLATING.md
├── practice/                   # Static adaptive practice quiz — HTML/JS/CSS + Python build.py + JSON banks
```

Each topic folder contains a named index file (e.g., `database-objects.md`, `advanced-tsql.md`) and numbered `.md` topic files.

Top-level files:

- `README.md` — public-facing entry point with badges, exam overview, and quick navigation. Rewrite when the blueprint date or major features change.
- `LICENSE` — MIT.
- `CLAUDE.md` — this file. Project conventions for AI assistants and contributors.
- `CONTRIBUTING.md` / `CONTRIBUTORS.md` / `CHANGELOG.md` — public-facing community files.
- `TRANSLATING.md` — translation conventions: BCP-47 locale codes, `i18n/<locale>/` mirror layout, priority order, currency policy. Translations must not alter English source files.
- `OBSIDIAN-SETUP.md` — optional setup notes for editing the guide in Obsidian.

## Practice Quiz (`practice/`)

A static browser-based quiz live at <https://kengio.github.io/dp-800-study-guide/>. Auto-deployed by `.github/workflows/deploy-practice.yml` on any push that touches `practice/**` or the source markdown.

- **Source of truth is markdown.** `practice/build.py` parses `certification/resources/practice-questions/*.md` and both `mock-exam/questions.md` + `mock-exam-2/questions.md` into `practice/data/*.json`. The deploy workflow re-runs `build.py` on every deploy, so the live site always reflects current markdown even if the committed JSON is stale.
- **Edit markdown, not JSON.** New questions, fixes, and renumbering go in the source `.md` files; the JSON in `practice/data/` is generated. Run `python3 practice/build.py` locally to refresh before committing.
- **Question format** — see [`practice/format.md`](./practice/format.md). The parser supports three heading formats, accepts `A.` or `A)` choices, and recognises both `**B. <choice>**` and `**Correct Answer: B**` styles inside the `> [!success]-` callout. Mock-exam domains are demarcated by `<!-- DOMAIN N: <name> (~M questions) -->` HTML comments; case-study sub-questions use `###` H3 headings under a `## Case Study: ...` H2.
- **Question `id` is stable across rebuilds.** Don't renumber questions inside a `.md` file without intent — it breaks learners' localStorage progress.
- **Same UX as the upstream Databricks study guide.** `index.html` + `app.js` + `styles.css` + `favicon.svg` were ported verbatim; only branding strings, the `CERTS` list, the `dp800-practice-` storage prefix, and the timer presets were swapped.

## Currency Policy

The exam blueprint is the source of truth. When Microsoft updates the [skills-measured list](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/dp-800):

- Update the **"What's New for the 2026 Exam"** callout at the top of `certification/dp-800-overview.md` with the new blueprint date and the diff vs. the prior version.
- Update the **"2026 Updates"** section in `certification/resources/final-review.md` to surface the highest-leverage new facts for exam day.
- Update the blueprint date badge in `README.md`.
- Mark questions that target newly added skills with a `*(YYYY update)*` suffix in the question heading so studiers can find them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kengio/dp-800-study-guide](https://github.com/kengio/dp-800-study-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
