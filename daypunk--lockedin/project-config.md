---
trigger: always_on
description: Project-level instructions, loaded automatically by Claude Code when
---

# CLAUDE.md

Project-level instructions, loaded automatically by Claude Code when
working in this repo. Read this first if you are picking up the
project in a fresh session. The file is designed to be self-contained
enough that you can resume without prior conversation context.

## What this is

LockedIn is a Claude Code plugin that organizes a user's experience
(career, meetings, projects, learning, decisions) into a typed
markdown ontology they own at `~/Documents/LockedIn/`, then renders
text artifacts (English resume, Korean cover letter) from the same
vault. Two stores by design.

- **Store A** is the user's input, structured into the vault under
  `~/Documents/LockedIn/experience/`.
- **Store B** is `~/Documents/LockedIn/outputs/`, the artifacts
  rendered from A.

This repo holds the engine. It contains the plugin manifests, skill
files, CLI helpers, and tests. Users' actual data lives at
`~/Documents/LockedIn/`, never in this repo.

Tagline: *Personal experience knowledge graph for Claude Code. Zero
learning curve.* Distribution is the Claude Code plugin marketplace
(`/plugin marketplace add daypunk/LockedIn`).

The lowercase identifier `lockedin` is used everywhere a machine
parses it (Python package, plugin name, slash command, env vars). The
brand display `LockedIn` is used in user-facing copy (README headers,
descriptions, sentences in prose).

## Current state

| Layer | State |
| --- | --- |
| Plugin marketplace | manifests at `.claude-plugin/marketplace.json` and `plugins/lockedin/.claude-plugin/plugin.json` |
| One-time setup wizard | `/lockedin:setup` at `plugins/lockedin/commands/setup.md` (HUD wiring, Q&A language, vault path) |
| Ontology | **v3**. 15 entity types, 15 edge predicates, JSON Resume / Schema.org / FOAF aligned. Per-type field contracts plus `aliases` (person, company) and `provenance` (every type). Edge domain and range enforced by `lockedin validate`. See `lockedin/ontology/schema.py`. |
| Render skills | `lockedin` (main), `lockedin-render-jaso` (calibrated, 5 pass + 5 fail fixtures), `lockedin-render-resume-en` (calibrated, 3 personas). All under `plugins/lockedin/skills/`. |
| HUD | `lockedin X.Y.Z │ 5h:NN% · wk:NN% │ experience: Nn · Me`. Pulls real utilization from Anthropic OAuth API, falls back to vault-only display when OAuth is unavailable. Same script in two places (`plugins/lockedin/scripts/hud.py` standalone, `lockedin/commands/hud.py` package). The standalone defers to the package when available. |
| Deterministic CLI | `install` (with `--setup-hud`, `--remove-hud`, lifecycle), `init --fixture FILE`, `ingest --dry-run`, `validate`, `migrate`, `experience`, `doctor`, `template`, `hud`. |
| Skill-only commands | `init` (interactive), `ingest` (smart), `render jaso/resume`, `query`. Typed in plain shell, the CLI prints a redirect (exit 3) and points at Claude Code. |
| OSS infrastructure | `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` (Contributor Covenant 2.1), `.github/ISSUE_TEMPLATE/{bug_report, feature_request, korean_reviewer_onboarding, config}.yml`. |
| Tests | 49+ passing across import, ontology, storage round-trip, install lifecycle, doctor, validate v3, init from fixture, template, ingest dry-run, hud, plus non-LLM jaso fixture validation. |

## Repo layout

```
.claude-plugin/marketplace.json     ← marketplace catalog
.github/
├── ISSUE_TEMPLATE/                 ← bug, feature, Korean reviewer, config
└── workflows/ci.yml                ← lint, test, language policy, leakage scan
plugins/lockedin/                   ← the plugin itself
├── .claude-plugin/plugin.json
├── commands/setup.md               ← /lockedin:setup wizard
├── scripts/
│   └── hud.py                      ← standalone HUD (no Python pkg required)
└── skills/
    ├── lockedin/                   ← main skill (SKILL/AGENTS/TOOLS.md)
    │   └── templates/experience/questions.yaml
    ├── lockedin-render-jaso/       ← Korean cover letter (Korean OK in skill files)
    │   └── RUBRIC + prompts + banned_phrases + reviewers + research-notes
    └── lockedin-render-resume-en/  ← English resume (us-tech-senior, mid, pm-product)
        └── RUBRIC + prompts + banned_phrases + research-notes

lockedin/                           ← Python package (optional CLI accelerator)
├── cli.py · config.py · __main__.py · __init__.py
├── ontology/{__init__,schema}.py   ← 15 entity types, 15 edge predicates, v3
├── storage/{notes,graph}.py        ← markdown read/write, edge derivation
├── ingest/{interview,markdown,pdf,docx,text}.py
├── render/_template.py
├── hud/oauth_usage.py              ← Anthropic OAuth credential reader + caller
└── commands/{install,doctor,validate,init,ingest_dry,migrate,experience,template,hud}.py

docs/                               ← architecture, spec, mappings, guides
├── architecture.md · ontology-spec.md · ontology-mapping.md
├── orchestration.md · hud.md · cli.md
└── adr/0001-viz-library.md         ← SUPERSEDED: render-graph removed in 1.1

tests/                              ← pytest suite + fixtures
├── test_smoke · test_storage_roundtrip · test_commands · test_hud · test_init_template_ingest
└── fixtures/jaso/{pass,fail}/      ← 5 pass + 5 fail synthetic golden fixtures (research-based)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daypunk/LockedIn](https://github.com/daypunk/LockedIn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
