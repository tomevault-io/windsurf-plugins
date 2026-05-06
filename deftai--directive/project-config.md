---
trigger: always_on
description: You are working inside the deft framework repository itself.
---

# Deft — Development Framework (deft repo)

You are working inside the deft framework repository itself.
Full guidelines: main.md

## First Session (deft development)

**Headless bypass**: If you have been dispatched with a specific task (e.g. cloud agent, CI agent, scheduled run), skip the onboarding checks below and proceed directly to your task. The onboarding flow is for interactive sessions only.

Check what exists before doing anything else:

**USER.md missing** (~/.config/deft/USER.md or %APPDATA%\deft\USER.md):
→ Read skills/deft-directive-setup/SKILL.md and start Phase 1 (user preferences)

**USER.md exists, PROJECT-DEFINITION.vbrief.json missing** (./vbrief/):
→ Read skills/deft-directive-setup/SKILL.md and start Phase 2 (project definition)

## Returning Sessions

When all config exists: read the guidelines, your USER.md preferences, and PROJECT-DEFINITION.vbrief.json, then continue with your task.

~ Run `skills/deft-directive-sync/SKILL.md` to pull latest framework updates and validate project files.

### Deft Alignment Confirmation

! At the start of each interactive session, after loading AGENTS.md, confirm to the user that Deft Directive is active. The confirmation must be unambiguous -- for example: "Deft Directive active -- AGENTS.md loaded."

! If the agent detects a context window shift or is asked "are you using Deft?", re-confirm alignment by stating that Deft Directive is active and AGENTS.md was loaded.

⊗ Begin an interactive session without confirming Deft alignment to the user.

Note: A true UI indicator (e.g. Warp status bar) is deferred to Phase 5. This is a behavioral rule only.

## Skill Completion Gate

! When a skill's final step is complete, explicitly confirm skill exit and provide chaining instructions if applicable. The confirmation must be unambiguous -- for example: "{skill-name} complete -- exiting skill." followed by what the user/agent should do next (e.g. wait for PR review, return to monitor, chain into another skill).

⊗ Exit a skill silently without confirming completion or providing next-step instructions.

## Before Improvising

- ! Before designing a multi-step workflow from scratch, scan `skills/` for an existing skill that covers the task — skills are versioned, tested, and encode lessons from prior runs
- ⊗ Improvise a multi-step workflow without first checking `skills/` for coverage

## Skill Routing

When user input matches a trigger keyword, read the corresponding skill:

- "review cycle" / "check reviews" / "run review cycle" → `skills/deft-directive-review-cycle/SKILL.md`
- "swarm" / "parallel agents" / "run agents" → `skills/deft-directive-swarm/SKILL.md` — chains to `deft-directive-review-cycle` at Phase 5
- "refinement" / "reprioritize" / "refine" → `skills/deft-directive-refinement/SKILL.md` — chains to `deft-directive-review-cycle` at exit
- "build" / "implement" / "implement spec" → `skills/deft-directive-build/SKILL.md`
- "cost" / "budget" / "pre-build cost" / "how much will this cost" → `skills/deft-directive-cost/SKILL.md`
- "setup" / "bootstrap" / "onboard" → `skills/deft-directive-setup/SKILL.md`
- "sync" / "good morning" / "update deft" / "update vbrief" / "sync frameworks" → `skills/deft-directive-sync/SKILL.md`
- "pre-pr" / "quality loop" / "rwldl" / "self-review" → `skills/deft-directive-pre-pr/SKILL.md`
- "interview loop" / "q&a loop" / "run interview loop" → `skills/deft-directive-interview/SKILL.md`
- "release" / "cut release" / "v0.X.Y" / "publish release" → `skills/deft-directive-release/SKILL.md` — operationalizes the `task release` / `task release:publish` / `task release:rollback` / `task release:e2e` surface (#74 + #716 safety hardening); re-uses the `skills/deft-directive-swarm/SKILL.md` Phase 6 Step 5 Slack announcement template

## Development Process (always follow)

### Implementation Intent Gate (#810)

- ! Run `task vbrief:preflight -- <path>` before any code-writing tool call or `start_agent` dispatch -- the gate exits 0 only when the candidate vBRIEF lives in `vbrief/active/` AND `plan.status == "running"`. The Taskfile target wraps `scripts/preflight_implementation.py` so the same invocation works whether deft is the project root or installed as a `deft/` subdirectory. The ONLY supported way to satisfy a non-zero exit is `task vbrief:activate <path>` (idempotent).
- ! Require an explicit action-verb directive (`build`, `implement`, `ship`, `swarm`, `run agents`, `start agent`) from the user before invoking the preflight gate or `start_agent` for implementation. When intent is ambiguous, ask one targeted question instead of inferring.
- ⊗ Infer implementation intent from lifecycle vocabulary ("do the full PR process", "start the work", "poller agents"), branching language, or workflow shape. Workflow-shape vocabulary is NOT authorization to spawn an implementation agent.
- ⊗ Treat affirmative continuation phrases (`yes`, `go`, `proceed`, `do it`) as implementation authorization unless the prior turn explicitly proposed implementation. Broad approval is not a substitute for an explicit action-verb directive.

**Before code changes:**
- ! Check `./vbrief/` lifecycle folders for existing scope vBRIEF coverage of the issue being fixed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deftai/directive](https://github.com/deftai/directive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
