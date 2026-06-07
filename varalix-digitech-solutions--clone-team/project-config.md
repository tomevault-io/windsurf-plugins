---
trigger: always_on
description: This skill was built with `skill-creator`. To iterate on quality, use its eval
---

# CLAUDE.md — clone-team skill repository

This repo **is** a Claude Code skill: `clone-team`. It orchestrates a team of
agents to clone any website into a pixel-perfect UI **and** produce thorough
architecture documentation. It's intended to be installed into
`~/.claude/skills/clone-team/` and published open-source.

If you're Claude working *on this repo* (developing the skill), read this. If
you're Claude *using* the skill to clone a site, the entry point is `SKILL.md`.

## What this skill is, in one breath

The **Manager** (the main-thread Claude running `SKILL.md`) gathers requirements
and credentials interactively, does recon + foundation, then launches a
**deterministic background `Workflow`** that enforces, per section:
`extract → spec → develop → full-regression-test → fix → re-test …` until the
**Tester** gate passes — with a **Backend Architect** documenting the system in
parallel. The whole run is **pausable and resumable** (even across sessions /
after usage-limit cutoffs) via a durable `state.json`.

The two deliverables, always: **(1) an exact UI clone**, **(2) `ARCHITECTURE.md`**.

## Why a Workflow (the core design decision)

The loop is a JS `Workflow` script, not Manager discretion, **so the Tester gate
cannot be silently skipped.** The script *is* the process: the Tester check is a
`while`-loop condition. This is the single most important property of the skill —
don't "simplify" it back into "spawn some agents and iterate," which is exactly
the failure mode (skipped tests, accepted "looks fine" sections) it prevents.

A Workflow runs in the background and can't talk to the user mid-run, so the
division is deliberate:
- **Interactive** (creds, requirements, clarifications, checkpoint sign-off,
  resume decisions, human final regression) → the **Manager** in `SKILL.md`.
- **Autonomous grind** (the build/test loop, backend docs) → the **Workflow**.

## File map

```
SKILL.md                          # the Manager playbook (entry point)
agents/
  frontend-developer.md           # builder persona (canonical)
  interaction-motion-analyst.md   # motion-spec author (canonical)
  motion-developer.md             # sequential motion polish pass (canonical)
  tester.md                       # the gate persona (canonical)
  backend-architect.md            # docs persona (canonical)
workflows/
  clone-build-loop.js             # the enforced loop (the engine)
references/
  orchestration.md                # team model + how to launch/steer the Workflow
  state-and-resume.md             # state.json schema, creds pattern, pause/resume/recovery
  extraction-playbook.md          # recon/extraction scripts + spec template
  motion-playbook.md              # motion taxonomy, state-matrix/token templates, drive-to-verify recipe
  backend-doc-template.md         # ARCHITECTURE.md structure
scripts/
  state.mjs                       # durable state CLI (init/status/mark-section/remaining/…)
  capacity.mjs                    # host-capacity probe -> recommended waveSize (check before launch)
  install-deps.sh                 # idempotent dep bootstrap: agent-browser CLI + companion skills (run at Preflight)
vendor/
  ui-pack/SKILL.md                # the vendored ui-pack wrapper skill (installer copies it to ~/.claude/skills)
commands/
  clone-status.md / clone-pause.md / clone-resume.md
evals/
  evals.json                      # skill-creator test cases
```

## Conventions & invariants (don't break these)

- **Single source of truth for personas.** The canonical personas are in
  `agents/*.md`. The Workflow embeds tight capsules of the same text but accepts
  `args.personas.{fe,motionAnalyst,motionDev,backend,tester}` overrides — the
  Manager reads the agent files and passes them in, so there's one source. If you
  edit a persona, edit the `agents/*.md` file.
- **Every agent loads `ui-pack` first** and verifies via `agent-browser`. This is
  baked into every persona; keep it there. The two motion agents additionally load
  `ui-animation` (degrading to `references/motion-playbook.md`).
- **Dependencies are self-bootstrapping; `ui-pack` is vendored, not external.**
  `ui-pack` is a thin **wrapper** skill (loads `clone-website`, `ui-ux-pro-max`,
  `impeccable`, `emil-design-eng` + points at the `agent-browser` CLI). It is
  **vendored at `vendor/ui-pack/`** so a public install never depends on a second
  repo. `scripts/install-deps.sh` is the idempotent bootstrap: it copies vendored
  `ui-pack` and git-clones the public constituents + `ui-animation`, and installs
  `agent-browser` via npm. **Skills install PROJECT-LOCAL by default** (into
  `$PWD/.claude/skills`) so a clone run never pollutes the user's global
  `~/.claude/skills`; `--global` opts into `~/.claude/skills`, `CLAUDE_SKILLS_DIR`
  overrides both. SKILL.md runs it at Preflight, from the project root. Don't make
  the agents depend on an un-vendored `ui-pack`, don't let the installer overwrite
  an already-present skill (it detects and skips), and don't revert the local
  default to global.
- **Two gates, never one:** Tester (in the loop) then Manager (final). Approved
  work only.
- **The motion track is additive and ordered — don't collapse it.** A dedicated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Varalix-Digitech-Solutions/clone-team](https://github.com/Varalix-Digitech-Solutions/clone-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
