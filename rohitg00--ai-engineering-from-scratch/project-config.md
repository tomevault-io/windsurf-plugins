---
trigger: always_on
description: Operating manual for contributors and AI agents touching this repo. Read it before opening a PR.
---

# AGENTS.md

Operating manual for contributors and AI agents touching this repo. Read it before opening a PR.

The repo is a curriculum, not a SaaS app. The lessons are the product. Every rule below keeps 435 lessons coherent over time.

---

## Philosophy

435 lessons. 20 phases. Every algorithm built from raw math before a single framework gets imported. You write backprop, the tokenizer, the attention mechanism, and the agent loop by hand in Python, TypeScript, Rust, or Julia. Then you run the same operation through the production library so the framework stops being a black box. The "Build It / Use It" split is the spine. Each lesson ships a reusable artifact you can plug into your daily workflow.

---

## Repo layout

```
phases/
  NN-phase-slug/
    NN-lesson-slug/
      docs/en.md              # lesson explainer
      code/                   # implementation + tests
      quiz.json               # 6 questions
      outputs/                # reusable artifact (skill / prompt / agent / MCP server)
README.md                     # public face; lesson counts auto-synced
ROADMAP.md                    # phase/lesson status
glossary/terms.md             # canonical term definitions
site/
  build.js                    # parses README + ROADMAP + glossary -> data.js
  data.js                     # generated; rebuilt by CI on main push
scripts/                      # automation
.github/workflows/
  curriculum.yml              # invariant + auto-sync workflow
```

---

## Hard rules

1. **One commit per lesson directory.** Never batch multiple lessons into one commit. A 10-lesson PR has 10 commits.
2. **Conventional commit subjects** ≤72 chars: `feat(phase-NN/MM): <slug>`. Body explains why, not what.
3. **Mermaid or SVG only** for diagrams. No ASCII / Unicode box-drawing.
4. **Every fenced code block needs a language tag.** Use `text`, `json`, `python`, `typescript`, `rust`, `julia`, `bash`, `console`, `mermaid`, `yaml` as appropriate.
5. **Original implementations only.** Don't cite external curriculum repos in docs, code comments, or commit text. Cite RFCs, official specs, and academic papers when they are the canonical source.
6. **Dependency allowlist** (see `Dependencies` below). Stdlib-first.
7. **Never commit generated files**: `catalog.json` is gitignored, `site/data.js` is rebuilt by CI, `package-lock.json` is never tracked.

---

## Dependencies

| Language   | Allowed                                                                  |
|------------|--------------------------------------------------------------------------|
| Python     | `numpy`, `torch`, `h5py`, `zstandard`, `safetensors`, stdlib              |
| TypeScript | `hono`, `zod`, `ws` (only when WebSockets needed), `@hono/node-server`, Node 20+ stdlib |
| Rust       | stdlib only (single-file `rustc --edition 2021`)                          |
| Julia      | `Random`, `Statistics`, `LinearAlgebra`, `Printf` (Julia stdlib)          |

If a finding suggests a banned dep, skip it with the reason "stays stdlib-first for educational clarity."

---

## Lesson contract

### docs/en.md frontmatter

```markdown
# <Title>

> <One-line hook>

**Type:** <Learn | Build | Reference>
**Languages:** <comma-list matching the main.* files in code/>
**Prerequisites:** <comma-list of upstream lessons, or "None">
**Time:** ~<estimate in minutes>

## Learning Objectives
- <4-6 bullet points starting with a verb>
```

The `**Languages:**` field must match the languages with a `main.*` file in `code/`.

### quiz.json schema

```json
{
  "lesson": "<dir-slug>",
  "title": "<Lesson Title>",
  "questions": [
    {"stage": "pre",   "question": "...", "options": ["a","b","c","d"], "correct": 0, "explanation": ""},
    {"stage": "check", "question": "...", "options": ["a","b","c","d"], "correct": 1, "explanation": ""},
    {"stage": "check", "question": "...", "options": ["a","b","c","d"], "correct": 2, "explanation": ""},
    {"stage": "check", "question": "...", "options": ["a","b","c","d"], "correct": 1, "explanation": ""},
    {"stage": "post",  "question": "...", "options": ["a","b","c","d"], "correct": 3, "explanation": ""},
    {"stage": "post",  "question": "...", "options": ["a","b","c","d"], "correct": 0, "explanation": ""}
  ]
}
```

Exactly 6 questions: 1 pre + 3 check + 2 post. `correct` is zero-indexed. The site renderer only understands this shape — legacy `q/choices/answer` schemas crash silently.

### code/

- Runs end-to-end and exits 0 on the canonical command for the language.
- Self-terminating demo. No infinite stdin loops, no hangs on missing API keys.
- 4-6 line header comment citing the lesson's `docs/en.md` path and any spec or RFC sources.

### code/tests/

- 5+ unit tests minimum.
- Runs via the language's stdlib runner (`python3 -m unittest discover`, `npx tsx --test`, Rust/Julia inline).

---

## Per-PR validation

Run locally before pushing:

```bash
python3 scripts/audit_lessons.py
python3 scripts/check_readme_counts.py        # advisory — CI fixes on merge

# For each lesson touched:
cd phases/NN-phase/MM-lesson/code
python3 main.py && python3 -m unittest discover tests -v   # or the lang equivalent
```

CI gates (`.github/workflows/curriculum.yml`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rohitg00/ai-engineering-from-scratch](https://github.com/rohitg00/ai-engineering-from-scratch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
