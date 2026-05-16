---
trigger: always_on
description: BA-Kit is a squad of **33 BA Specialists** for Requirements Engineering.
---

# BA-Kit Antigravity — Agent Rules

## Project Overview

BA-Kit is a squad of **33 BA Specialists** for Requirements Engineering.
Knowledge engine: BM25+ over 831 entries across 23 domains.
Templates: 14 BA document templates in `.agent/templates/`.

## Agent Squad (invoke via /ba-*)

| Agent | Role |
| :--- | :--- |
| `/ba-master` | Dispatcher — routing & squad planning |
| `/ba-identity` | Chief of Staff — persona & stakeholder mapping |
| `/ba-elicitation` | Journalist — funnel questioning |
| `/ba-writing` | Architect — user stories, Gherkin |
| `/ba-validation` | QA Lead — visual QA, edge cases |
| `/ba-traceability` | CCB Secretary — RTM, impact analysis |
| `/ba-nfr` | SRE Architect — ISO-validated NFRs |
| `/ba-process` | Lean Master — process mapping, waste analysis |
| `/ba-prioritization` | Product Manager — MoSCoW, RICE, WSJF |
| `/ba-solution` | Investor — ROI/NPV analysis |
| `/ba-conflict` | Mediator — Harvard negotiation, ADR |
| `/ba-export` | Publisher — compliance, formatting |
| `/ba-metrics` | Data Scientist — SPC charts, Cpk stats |
| `/ba-root-cause` | Investigator — 5 Whys, Fishbone, Pareto |
| `/ba-innovation` | R&D Scientist — A/B testing, hypothesis design |
| `/ba-strategy` | Strategist — PESTLE, SWOT, BMC |
| `/ba-facilitation` | Facilitator — workshop design, ODEC |
| `/ba-systems` | Systems Analyst — stocks & flows, leverage points |
| `/ba-agile` | Agile Analyst — story mapping, MVP, estimation |
| `/ba-jira` | Jira Bridge — story→ticket transport |
| `/ba-confluence` | Confluence Bridge — markdown→XHTML publishing |
| `/ba-test-gen` | QA Architect — AC → 7-category test cases |
| `/ba-quality-gate` | Quality Officer — 8-dimension quality scoring |
| `/ba-consistency` | Integration Auditor — cross-artifact alignment |
| `/ba-auditor` | Chief Auditor — project health dashboard |
| `/ba-questioning` | Critical Thinker — Paul-Elder framework, assumption surfacing |
| `/ba-communication` | Communicator — status reports, executive summaries |
| `/ba-ux` | UX Analyst — persona, journey mapping, usability testing |
| `/ba-data` | Data Analyst — ERD, data dictionary, DFD, migration |
| `/ba-change` | Change Manager — ADKAR, training needs, go-live |
| `/ba-business-rules` | Rules Engineer — decision tables, rule catalog |
| `/ba-diagram` | Visual Architect — Mermaid v11 (24+ types) |
| `/ba-wiki` | Knowledge Curator — wiki query, living documentation |

## Key Commands

```bash
# Search knowledge base (831 entries, 23 domains)
python3 .agent/skills/ba-kit-search/scripts/ba_search.py "<query>"

# Check US/AC coverage
python3 .agent/skills/ba-kit-search/scripts/coverage_checker.py <file>
```

## Development Principles

- YAGNI / KISS / DRY
- System 2 reflection before every output
- Verify math with Python, verify links with Grep, verify standards with WebSearch
- Never hallucinate file contents — check with Grep/Read

## Documentation Structure

```
.agent/skills/ba-*/SKILL.md   — 44 agent skill files + 2 connectors + 1 meta
.agent/templates/              — 14 BA document templates
.agent/scripts/                — BM25+ search, ba_e2e_test, sprint-spine helpers
.github/workflows/e2e-skills.yml — CI quality gate (Phase 08)
docs/                          — guides, cheat sheet, prompt library, sprint-spine
outputs/mini-app-cham-cong/    — BA showcase reference (12 modules, 53 US)
ebooks/                        — 7 synthesized BA ebooks
```

---

## 2. Documentation Quality Rules

### 2.1 Template Completeness
Every module README must contain:
- [ ] Process Flow diagram (`graph TD`)
- [ ] Use Case diagram (`graph LR`)
- [ ] Metadata table
- [ ] User needs table
- [ ] NFR section

### 2.2 Test Suite Standards
Every test-cases.md must have:
- 7-column format: TC-ID | Category | Precondition | Steps | Input | Expected Result | Priority
- BVA (Boundary Value Analysis) section for numeric fields
- All 7 categories: Happy, Edge, Error, Security, Concurrency, Data, Performance
- Coverage Summary table

### 2.3 Batch Generation Limit
- Generate complex artifacts (test suites, API specs) for **max 3 modules** per session
- Quality degrades with larger batches due to context pressure

---

## 3. Confluence Sync Rules

### 3.1 Environment
- `CONFLUENCE_BASE_URL` and `CONFLUENCE_PAT` must be set in `.env`
- Target space: `CVH` (C-Vision Hub)
- Target instance: CTS Knowledge Hub (kms.cmcts.com.vn) — Data Center

### 3.2 Local vs Confluence Formats
- **Local markdown**: Keep native languages (`json`, `gherkin`, `mermaid`) — VSCode/GitHub renders them
- **Confluence upload**: Use `confluence_xhtml.py` which auto-maps to DC-safe equivalents
- Never manually construct XHTML with `language="json"` or `language="gherkin"`

### 3.3 Post-Upload Checklist
After every bulk upload:
1. Run `validate_rendered_pages()` scan
2. Verify 0 broken pages
3. Spot-check 2-3 pages visually via browser

---

## 4. Agent Skill References

| Skill | Reference Files |
|-------|----------------|
| confluence-connector | `references/confluence-dc-rendering-rules.md`, `scripts/confluence_xhtml.py` |
| ba-diagram | HARD RULES R1-R5 in SKILL.md |
| ba-test-gen | 7-category system, 7-column output format |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olbboy/BA-Kit](https://github.com/olbboy/BA-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
