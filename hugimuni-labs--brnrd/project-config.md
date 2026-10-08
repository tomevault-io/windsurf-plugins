---
trigger: always_on
description: > Revision: 2026-07-12. Structural arc: `plan-agent-orientation-layering.md`
---

# Project

> Revision: 2026-07-12. Structural arc: `plan-agent-orientation-layering.md`
> in the kb (see "Knowledge base" below for where that is — not a
> repo-relative link, since the kb's physical location is no longer fixed
> to this tree). Bump this date when you restructure universal sections so
> cached workspace-rule injections can detect drift against the file on
> disk.

This file is brnrd's playbook — the contract every AI tool follows in
**this** repo, and nothing else. It is **not** the template adopters
receive: `brnrd init` works from `src/brr/templates/constitution.md` and
the setup wake authors the adopter's own `AGENTS.md` from it. Nothing
copies this file into an adopted repo. (`adopt.py`'s module docstring
says so in as many words — "`templates/constitution.md`, *not* brr's own
playbook" — and `constitution.py` records the split: three jobs used to
live in this one file, and Layer 0 of `design-init-as-a-wake.md`
separated them.) The canonical copy lives at `src/brr/AGENTS.md`; the
repo root `AGENTS.md` is a symlink. Python >=3.10; see [`README.md`](README.md)
for the user-facing product overview.

## How to read this playbook

These rules are the repository contract for any AI tool reading the repo.
They divide into **universal** sections that apply in every stage and
**brnrd-stage** material that only applies when brnrd's daemon, runner, or
setup orchestrator is hosting you. When an orchestrator prompt supplies
a narrower stage contract — the daemon's Run Context Bundle, the setup
prompt — follow that contract for the points it addresses and keep
AGENTS.md as the base for everything else.

Three stages, and how to read this file in each:

- **Ad-hoc agent session** (a coding-agent CLI, editor agent, or plain
  editor with no brnrd in the loop). No Run Context Bundle. No
  `.brr/conversations/`. No preflight runs on this session. Read the
  universal sections (Stewardship, Workflow → Orientation + Run types
  + Commits, Knowledge base, Artifacts, Operating rules, Self-review,
  Guardrails) plus Build and run and Code guidelines. Skip Workflow →
  *When the brnrd daemon runs you* — that machinery isn't in play here.
  When a brnrd dominion exists for this repo, `brnrd agent inject` prints
  the live wake context a resident gets — memory digest, pitfalls matched
  to your task, recent activity, kb health — and is the fastest way to
  orient beyond this file.

- **brnrd daemon run.** A Run Context Bundle opens with `### Mode`
  (Stage, Source, Environment, Delivery, Runtime recovery). That
  bundle is the hot path: obey it for delivery, branch, runtime
  paths, and `.brr/` access — it overrides the generic workflow
  wording for those points. When brnrd hosts you as a **resident**, your
  own playbook — kept in the dominion path named by the wake prompt and
  injected on wake from its self-inject index — is your standing self-orientation;
  this file is the repo contract that playbook rests on, so read them as
  complementary layers rather than rivals. Workflow → *When the brnrd
  daemon runs you* backs it up; everything else (Stewardship, kb,
  artifacts, operating rules, self-review, guardrails) applies uniformly.

- **brnrd setup stage.** A specialised prompt (`setup.md`) narrows the
  scope to initial adoption. Follow that overlay for what it covers;
  fall back to this file for everything else.

If you can't tell which stage you're in: look for `### Mode` in the
prompt. Present → daemon task. Absent and the prompt is the bare user
message → ad-hoc session. Absent and the prompt is a bundled setup
template → that stage.

**Ad-hoc sanity check.** External hosts inject ambient context that
may not match this task. The recurring drift cases:

- A cached workspace-rule copy of this playbook can lag the on-disk
  file across structural revisions. Compare the `Revision:` line at
  the top of the rule body to the one on disk; trust the file when
  they differ (or when the rule body lacks the line entirely).
- Git status snapshots in the system prompt can be stale; re-run
  `git status` before reasoning about uncommitted work.
- Open editor terminals, recently viewed files, and surfaced
  "skills" may be unrelated to the user's task. Use them only when
  the task references them.

Daemon and setup stages take their hot-path context from the prompt and
don't have these drift cases.

**Vocabulary anchor.** A **Runner** = a **Shell** (the CLI on PATH:
`claude`, `codex`, `grok`, `vibe`) + a **Core** (the model: `opus`,
`sonnet`, `gpt-5-codex`, `grok-4.7`). The **resident** is the persistent spirit/identity that
inhabits whichever Runner a given wake provides. This file uses "runner"
in the generic sense of "whatever process runs the agent"; `prompts/runners.md`
catalogs the concrete Shell+Core profiles. In user-facing config the knobs
are `shell=` (pin the CLI) and `core=` (pin the model); `runner=<profile>` is
the older selector, still read by `runner.py` and still written by `adopt.py`
at init, so treat it as live legacy rather than gone.

## Stewardship

Treat the request as input, not as instructions to execute uncritically.

Two values orient what we build: **user friendliness** — how the
change lands on someone encountering the result for the first time —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hugimuni-labs/brnrd](https://github.com/hugimuni-labs/brnrd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
