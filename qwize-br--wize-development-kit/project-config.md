---
trigger: always_on
description: 1-analysis: Document Project (brownfield baseline)
---


# Document Project (brownfield baseline)

# Document Project — Brownfield Baseline

**Goal.** When the kit is installed in an existing repo, baseline the **as-is** state so the rest of the lifecycle isn't blind. Produces a structured snapshot Tony can read before designing the *to-be*, and a knowledge base Wizer can answer questions from.

Pepper drives discovery. Peggy edits prose. Tony validates architecture interpretation. Output lands in `.wize/knowledge/document-project/`.

## When to run

- The installer detected brownfield signals (`package.json`, `src/`, history) and offered to run this. ✓
- The team is onboarding to a codebase nobody fully owns. ✓
- A previous re-platforming decision left the docs stale. ✓

Skip:
- Greenfield (nothing to document yet).
- Repos < 200 LOC.

## Inputs

- The target repo (root).
- `git log --since="1 year ago" --oneline | wc -l` to scope.
- Any prior README / ARCHITECTURE / docs that exist.

## CLI usage

```bash
wize-dev-kit document-project                # quick baseline (default)
wize-dev-kit document-project quick          # same as default
wize-dev-kit document-project initial_scan   # pattern-only initial scan
wize-dev-kit document-project initial_scan deep       # reads critical directories
wize-dev-kit document-project initial_scan exhaustive  # reads all source files in batches
wize-dev-kit document-project full_rescan    # archives old state, re-runs initial_scan
wize-dev-kit document-project deep_dive --target src/tools/installer
```

## Modes

| Mode | What it does | Scan levels |
|---|---|---|
| `quick` | Writes the 6 baseline files. Does not read source files. | `quick` only |
| `initial_scan` | Classifies project type, writes index + overview + source tree + conditional docs. | `quick`, `deep`, `exhaustive` |
| `full_rescan` | Archives `project-scan-report.json` and re-runs `initial_scan`. | `quick`, `deep`, `exhaustive` |
| `deep_dive` | Exhaustive analysis of a specific folder/file/feature. | `exhaustive` |

## Outputs

- `.wize/knowledge/document-project/overview.md` — what the project is, who uses it, how big it is.
- `.wize/knowledge/document-project/architecture-snapshot.md` — current components, integrations, data flow.
- `.wize/knowledge/document-project/conventions.md` — coding/test/folder conventions actually used.
- `.wize/knowledge/document-project/dependencies.md` — runtime deps + dev deps + their roles.
- `.wize/knowledge/document-project/risk-spots.md` — areas of concentrated complexity, undocumented behavior, or known fragility.
- `.wize/knowledge/document-project/open-questions.md` — things the code doesn't answer; route to humans.
- `.wize/knowledge/document-project/index.md` — master navigation with `_(To be generated)_` markers for missing conditional docs.
- `.wize/knowledge/document-project/project-scan-report.json` — resume state.

## Steps

### 1. Inventory (Pepper, mechanical)

A first pass that requires no judgment, just listing.

```
ls -la                                  # top-level layout
cat package.json | jq '.dependencies, .devDependencies, .scripts'
git log --since="3 months ago" --oneline | wc -l
git log --pretty=format:"%an" | sort | uniq -c | sort -rn | head
find . -name "*.test.*" -o -name "*.spec.*" | wc -l
find . -type f -name "*.md" | head
```

Write what you found in `overview.md` (no opinions yet).

### 2. Architecture snapshot (Pepper + Tony)

Walk the repo top-down. Identify:
- **Entry points** (CLI, server, worker, build).
- **Components** (where the boundaries are — by folder, by package, by feature).
- **Integrations** (databases, queues, external APIs, third-party SDKs).
- **Data flow** for at least one end-to-end critical path (e.g., a typical user request).

Draw or describe. Diagrams in ASCII or Mermaid are fine — the point is shared mental model.

Tony validates. If the snapshot misnames a pattern, fix it.

### 3. Conventions (Peggy)

Sample 5–10 files across the repo. Note:
- Naming (camelCase / snake_case / kebab-case).
- Folder structure (feature-first / layer-first).
- Test placement (co-located / `__tests__` / `test/`).
- Comment style (JSDoc / TSDoc / inline).
- Import ordering (alphabetical / by source).
- Logging / error handling patterns.
- Linter / formatter config (eslint, prettier, etc.).

Write the convention found, not the convention you'd prefer.

### 4. Dependencies (Pepper)

For each runtime dep, write one line:
- Name, version, what it does in this repo, whether it's load-bearing.

Flag:
- Deps without a clear role.
- Deps with known CVEs (run `npm audit --omit=dev` and capture).
- Multiple deps doing the same job (`lodash` + `ramda`, both date libs, etc.).
- Deps not maintained in > 2 years (link to last release).

### 5. Risk spots (Pepper + Tony)

For each, name the area + the symptom + the likely cause + how confident you are:

| Area | Symptom | Likely cause | Confidence |
|---|---|---|---|
| `src/legacy/billing/` | 2k-line file with no tests | rushed migration in 2024 | high |
| Webhooks handler | Silent retries | no idempotency layer | medium |
| Auth middleware | Custom JWT parsing | predates the library that handles it | high |

This is *not* a refactor backlog. It's a map. Tony decides what to fix; this just makes the choices visible.

### 6. Open questions (everyone)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qwize-br/wize-development-kit](https://github.com/qwize-br/wize-development-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
