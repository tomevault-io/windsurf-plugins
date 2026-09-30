---
trigger: always_on
description: This repository is __APP_NAME__'s GTM engine: knowledge, infrastructure, and run history
---

# __APP_NAME__ GTM

This repository is __APP_NAME__'s GTM engine: knowledge, infrastructure, and run history
for the whole go-to-market, managed as code. It follows the Manifest framework
conventions (Manifest, by Cargo).

The knowledge layers ship empty, not populated. Every repeating artifact has a
`_template.md` beside it to copy (`cadence/log/`, `cadence/weekly/`,
`initiatives/`, `outputs/`, each `context/<domain>/`, `evals/`), and the four
singletons that must exist by name (`plan/company-plan.md`, `plan/strategy.md`,
`plan/outcomes.md`, `cadence/carryover.md`) ship as skeletons: the headings and
the rules, none of the content. Nothing here is another company's data, so
there is nothing to find and delete before you start. `infra/` ships empty by
design too: add resources with `cargo-ai cdk add cookbook/<name>` or by writing
them.

## Layers

| Layer | Directory | What it is |
| --- | --- | --- |
| Plan | `plan/` | The destination: the one goal, the strategy, and the outcomes that must become true. Each outcome names what it becomes in `infra/`. |
| Context | `context/` | What the company knows: the durable GTM brain every agent reads before acting. |
| Initiatives | `initiatives/` | The big rocks: bounded efforts with an owner, a deadline, and success criteria checkable as true or false. |
| Cadence | `cadence/` | The operating rhythm: weekly plans, daily logs, and the carryover file that keeps dropped balls visible. Knowledge says what is true; cadence says what matters now. |
| Skills | `.claude/skills/` | What agents know how to do: procedures for operating this specific repo. |
| Infra | `infra/` | What runs in production: the deployed engine, declared in TypeScript, reconciled by the Cargo CDK. |
| Scripts | `scripts/` | What you run by hand: imperative glue for runtime-only surfaces the CDK cannot declare yet. |
| Evals | `evals/` | What keeps it honest: promptfoo regression suites for agent prompts, run with `npm run eval`. |
| Outputs | `outputs/` | What happened: the append-only archive that gives agents accumulating memory and humans an audit trail. |
| Scratch | `scratch/` | Yours alone: gitignored personal space with no conventions. Shared layers must never reference it. |

Dependency direction is strict: infra reads context, skills read everything,
outputs are written by everything and read only as memory. Nothing depends on
outputs.

## Read order

Before acting: `plan/` (where we are going) then `cadence/` (what matters this
week) then `context/` (what we know). Then the layer you are changing. An agent
that writes copy without reading `context/` will sound like a generic bot.

## Working in each layer

- `plan/` holds three files and no more. Every outcome carries an owner, a
  measure, and a `Becomes:` naming the play, agent, or tool in `infra/` that
  makes it real. An outcome with no `Becomes:` is a wish.
- `initiatives/` is one file per bounded effort, from `_template.md`. Success
  criteria must be checkable true or false. The log is append-only.
- `cadence/` is `weekly/YYYY-Www.md`, `log/YYYY-MM-DD.md`, and `carryover.md`.
  Use the `cadence` skill (plan the week, log the day, review the week). An
  item in carryover for three weeks is a decision being avoided, not a task:
  escalate it.
- `context/` holds markdown with YAML frontmatter (`title` and `description`
  required). Domains are fixed folders (icp, persona, motion, ...); each has a
  `_template.md`. Cross-reference other files with `references:` in frontmatter
  or `[[domain/slug]]` wikilinks. Run `npm run lint:context` before committing.
- `infra/` is a self-contained Cargo CDK project. Importing a `.ts` file IS
  registration; the directory layout is convention. NEVER edit
  `infra/cargo.state.json` by hand. Secrets come from the environment via
  `secret()` and `env()`; never commit values.
- `scripts/` is for one-off imperative operations against the workspace
  (memories, users, content libraries). If the CDK can declare it, it belongs
  in `infra/` instead.
- `evals/` uses promptfoo. When you change a system prompt in `infra/`,
  update or extend the matching suite in `evals/` and run `npm run eval` (needs
  an `OPENAI_API_KEY`). It is not wired into CI: add a workflow when the suites
  are worth gating on.
- `outputs/` entries are directories named `outputs/YYYY-MM-DD-<slug>/`, each
  with a `README.md` carrying an **`outcome:`** field (meetings, replies,
  pipeline attributed, or "none" with a reason). That field is what lets motions
  be ranked by results instead of opinions. Read recent entries for memory
  before starting related work. Never rewrite an existing entry.

## Workflow

1. Change `context/` or `infra/` on a branch. Before opening the PR, run
   `npm run lint` if you touched any prose, and `npm run typecheck` if you
   touched `infra/` or `scripts/`. Both also run in CI.
2. CI runs `cargo-ai cdk plan` and posts the diff as a comment. Review it the
   way you would review a terraform plan.
3. Merging to main deploys. Never run `cargo-ai cdk deploy` locally against
   production. Local deploy is denied by convention and hard-blocked per agent:
   Claude Code (`.claude/settings.json`), Cursor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getcargohq/cargo-manifest](https://github.com/getcargohq/cargo-manifest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
