---
trigger: always_on
description: Guidance for agents working in this repository.
---

# CLAUDE.md

Guidance for agents working in this repository.

## What this repository is

`humane` — a design *method* cycle packaged as a Claude Code plugin and its own
single-entry marketplace, portable to other agents. It covers the work around an
interface: what to build (`jtbd`), for whom (`persona-review`,
`respondent-panel`), under which system (`design-tokens`) and constraints
(`layout-rules`), in which words (`ux-writing`), audited (`nielsen-heuristics`)
and proven (`before-after`). Ring 2 (`brandkit`, `brand-illustrate`) handles
visual identity.

Skills live in `humane/skills/<name>/SKILL.md`; the bundled agent is
`humane/agents/synthetic-respondent.md`. Bump `version` in
`humane/.claude-plugin/plugin.json` when plugin users should receive an update.

## Rule ownership

Each rule lives in exactly one skill. Other skills reference it by skill name in
backticks and name only the handoff — never restate or override it.

| Skill | Owns |
| --- | --- |
| `setup` | Environment and configuration: the five settings, their resolution order, dependency checks, and the install commands for each gap. Owns no method rules. |
| `jtbd` | The corpus: jobs, switch forces, ODI outcomes, evidence ledger, granularity and jargon gates. Every downstream claim cites it. |
| `persona-review` | Expert stakeholder critique of a **document** that is meant to be studied |
| `respondent-panel` | Gut reactions from strangers to a **user-facing artifact**, in isolated contexts. Never rewrites. |
| `design-tokens` | The token set: DTCG structure, resolution, CSS/DESIGN.md export, the brand block, and **all color measurement and remediation** (`tokens contrast`) |
| `layout-rules` | Structural and interaction defect classes for tool/dashboard UIs, and the de-slop avoid-list |
| `ux-writing` | The source wording of every user-facing string, and what each string must accomplish — including documentation prose (README, docs, release notes) |
| `nielsen-heuristics` | Formal usability inspection against the 10 heuristics, with severity and evidence locators |
| `walkthrough` | Task completion — whether a specific person with a specific job can get through a specific interface |
| `review` | Orchestration only: scope, mode, domain order, consolidation, coverage honesty, and the single final verdict. Owns no domain rules. |
| `before-after` | The felt transformation claim, and the proof that a change worked |
| `brandkit` | Identity **exploration** — competing directions for a brand that does not exist yet |
| `brand-illustrate` | Asset **production** under an existing token contract |

Cross-boundary cases, resolved:

- `layout-rules` rule 12 states that text must clear the contrast bar;
  `design-tokens` owns measuring the pair and changing the color. A review that
  eyeballs contrast instead of running `tokens contrast` is doing it wrong.
- `ux-writing` owns what a string says; `layout-rules` owns whether the layout
  has room for it once translated, and how it renders.
- `nielsen-heuristics` judges whether the flow works; `layout-rules` judges
  whether the implementation contains known defect classes. An issue that is
  both is reported once, by the skill that owns the underlying rule.
- `respondent-panel` reports how copy lands; `ux-writing` performs the rewrite.
  Respondents never suggest alternatives — that is the point of them.
- `brandkit` runs before a token set exists and hands off into it;
  `brand-illustrate` runs after one exists and reads from it.
- `persona-review` and `respondent-panel` differ by artifact and by attention:
  a document studied by an expert vs. an artifact glanced at by a stranger.
- `nielsen-heuristics` inspects an interface against principles; `walkthrough`
  attempts a task on it. An interface can pass every heuristic and still be
  impossible to get through, and vice versa — run both, report each finding once
  under the skill that owns it.
- `review` never restates a rule. When it needs one, it names the owner. A rule
  written into `review` is a bug.

## What we deliberately do not cover

Typography mechanics, motion recipes, OKLCH palette construction, and
accessibility engineering depth are **not** in scope. When those matter, defer
to `interfaces` (`better-typography`, `better-ui`, `better-colors`,
`better-accessibility`, `better-layout`, `better-writing`) if installed, or to
`impeccable` for post-build polish. A review that cannot cover a domain says
**Not reviewed** and names what is missing; it never improvises the rules from
memory or claims holistic coverage it did not have.

## Authoring conventions

- **Frontmatter `description` is the discovery surface.** One sentence on what
  the skill does, then "Use when…", then a `Triggers on …` keyword list. Update
  the triggers whenever scope changes.
- **Progressive disclosure.** `SKILL.md` is the entry point and should stay
  readable in one sitting; put recipes, lookup tables, and long-form depth in
  `references/*.md` and route to them from a Quick Reference table. Add a
  reference file only when it carries depth beyond the principle statements, not
  to restate them at length.
- **Prescribe exactly, and say why.** "Always `scale(0.96)`, never below `0.95`
  — anything lower reads as exaggerated" beats "use a subtle scale". Where a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glebis/humane-agentic-design](https://github.com/glebis/humane-agentic-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
