---
trigger: always_on
description: validates the BC app build, `azure-integration-validator` validates the Azure
---

# AGENTS.md

The contract between you and every agent that works in this repository. This is
the canonical guidance file. `CLAUDE.md` imports it so Claude Code loads it
automatically; other agent tools read `AGENTS.md` directly.

## What this repo is

A community-maintained Business Central integration repository built on
[AL-Go for GitHub](https://github.com/microsoft/AL-Go), packaged with a curated
set of AI agents and skills for AL development, and driven by Spec-Driven
Development.

## How we build here: Spec-Driven Development

**The spec is the brain, the agent is the muscle.** You do not write production
AL until an approved spec exists for the feature. Specs are durable artifacts
under `specs/`; a chat session is not. The developer architects the decisions in
the spec; the agent executes them.

### The constitution (read first, every session)

- `specs/brief.md` — customer requirements and business processes (the what and why).
- `specs/tech-design.md` — implementation strategy: which standard BC modules to
  reuse, and where custom code is genuinely needed.
- `specs/roadmap.md` — ordered list of features with status.
- `AGENTS.md` (this file) — conventions, quality gates, and the workflow.

### The per-feature loop

For each feature: **Feature Spec -> Plan -> Implement -> Test -> Docs -> Merge**,
then replan the next feature. Use one chat session per feature, and start it by
reading the constitution.

1. **Spec.** Draft `specs/features/<id>/spec.md`: the problem, the users, scope,
   acceptance criteria, and what is out of scope. Then run `/al-spec-feature` to
   review it against the constitution and confirm the acceptance criteria are
   testable. Stop for human review. No code yet.
2. **Plan.** Draft `plan.md` and `tasks.md` next to the spec: the BC objects to
   add or extend, the object ID range, the data model, integration points, and an
   ordered task list. Then run `/al-plan-feature` to review them against the
   approved spec. Stop for human review.
3. **Implement.** Work the tasks in order, applying the house rules and citing
   BCQuality. Then run `/al-implement-feature` to review the implementation
   against its spec, plan, and tasks.
4. **Test.** Building the app triggers the post-build hook, which prompts the
   mandatory verifier agents and a BCQuality review. Resolve every finding.
5. **Docs and Merge.** Update the feature docs, tick the roadmap item, open a PR.

### Bootstrapping

At the start of a project (or when the high-level requirements change), draft
`brief.md`, `tech-design.md`, and `roadmap.md`, then run `/al-spec-init` once to
review the constitution for completeness and consistency before any feature work.

## Layout

- `specs/`: the constitution and per-feature specs (the source of truth for what to build).
- `.AL-Go/`, `.github/workflows/`: AL-Go for GitHub build, test, and release pipelines.
- `.claude/agents/`: focused reviewer and verifier subagents for AL code.
- `.claude/skills/`: invocable review skills, including the SDD review gates above.
  Most are thin pointers: the authoritative content lives in BCQuality
  (the single source of truth) as `action-skill` (review) files under
  `.claude/bcquality/custom/skills/<category>/`, and each pointer mirrors the
  upstream skill's frontmatter (the BCQuality `kind`/`id`/`version`/`title`/
  `inputs`/`outputs` and dimension schema) plus the `name`/`description` the
  runtime needs to discover and trigger it.
  Edit the skill upstream in EquerraNZ/community-BCQuality and re-vendor, not the
  pointer. `al-code-review` is the exception: it stays a full local skill because it
  carries this project's house rules.
- `.claude/bcquality/`: a vendored, Microsoft-authored subset of the
  [BCQuality](https://github.com/EquerraNZ/community-BCQuality) knowledge corpus.
  Plain committed files; there is no submodule to initialise. The `custom/skills/`
  layer holds this project's reusable `action-skill` review skills; the runtime pointers in
  `.claude/skills/` defer to them.
- `al.code-workspace`: opens the workspace and defines the agent task playlists.

## BCQuality: the knowledge source agents cite

BCQuality is a remedial, machine-readable knowledge base for BC. Agents cite it
so that findings are backed by a vetted rule rather than paraphrased from memory.
Read [`.claude/skills/bcquality-integration/SKILL.md`](.claude/skills/bcquality-integration/SKILL.md)
for the full contract. The essentials:

- **Where it lives.** `.claude/bcquality/`. Knowledge files are under
  `.claude/bcquality/<layer>/knowledge/<domain>/`. The Microsoft layer vendors
  domains `performance`, `privacy`, `security`, `style`, `testing`, `ui`,
  `upgrade`. The `community/` layer adds `performance` and `security` rules, and
  the `custom/` layer adds `api`, `integration`, `operations`, `performance`,
  and `process` knowledge plus review and testing skills.
- **How it is consumed.** An agent invokes `.claude/bcquality/skills/entry.md`
  first. Entry returns a dispatch record naming the action skill(s) to run.
  Action skills follow the four-step pattern (Source, Relevance, Worklist,
  Action) defined in `.claude/bcquality/skills/do.md`, reading `read.md` on demand.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EquerraNZ/community-integration](https://github.com/EquerraNZ/community-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
