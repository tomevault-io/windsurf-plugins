---
trigger: always_on
description: <!-- ledger-mandatory:v2 — MUST STAY AT THE TOP OF THIS FILE. Do not move it down. -->
---

<!-- ledger-mandatory:v2 — MUST STAY AT THE TOP OF THIS FILE. Do not move it down. -->
> # 🛑🧾 STOP — EVERY AGENT MUST READ THE LEDGER, AND MUST RECORD EVERY ERROR
> ## This is not optional, and it is the first thing you do.
>
> **Before you write anything in this repo: read `LEDGER.md`.** If this repo has none, read
> **`coreflux-ai-os/brain/LEDGER.md`** — the cross-system one. It is a list of mistakes that were
> already paid for once. **An agent that skips it makes them again.**
>
> **⚠️ WHY THIS BANNER IS AT THE TOP AND NOT FURTHER DOWN.** During the client-site factory build,
> the same mistakes kept recurring for weeks. The ledgers existed the whole time — but they were
> **buried, unreferenced, and optional**, so agents never opened them and the system never learned.
> **A ledger nobody is obliged to read is decoration.** That is why this sits above everything else.
>
> ### 1. READ — before your first edit
> The ledger tells you which approaches are already known to fail here. Reading it costs a minute;
> repeating an entry costs the whole task, and sometimes live work.
>
> ### 2. RECORD — every error, the moment it happens. No exceptions.
> **Log it even if:** you already fixed it · it seemed small · nobody noticed · it was your own
> mistake · it was embarrassing · it was "just" a typo in a script. **Especially then.**
> An error that isn't written down is an error the machine makes again on the next run.
>
> - **THE LAW: "A fix is not a fix. A GUARD is a fix."** An entry closes ONLY when a permanent
>   automated check makes that failure impossible to repeat *silently*.
> - `FIXED + GUARDED` = closed · `FIXED + RULE` = **NOT closed** (a rule is a suggestion under
>   pressure) · `OPEN`. Never write a status you haven't earned.
> - **One entry per ROOT CAUSE, never per symptom.** If it reads like a symptom, dig again.
> - **Any CRITICAL, or anything seen 3 times → a full incident report** (background · condition ·
>   5-whys · countermeasure · verification · follow-up). That is the **andon cord**: the line stops
>   and explains itself, or the system never compounds.
> - **Never delete an entry, never squash the file, never gitignore it.**
>
> ### 🛑 The four failures that have cost the most. Do not repeat them.
> 1. **`git add -A`** — in a script or by hand. **Stage explicit paths.** (L-001/L-002 — 3× in one day.)
> 2. **Editing by character offset between two landmarks** — it slices through markup. Operate on
>    whole elements; assert structural balance before *and* after. (L-003)
> 3. **Trusting a detector that was never negative-tested** — inject a violation, watch it fail,
>    restore. **Fix the detector, never loosen the check.** (L-004)
> 4. **`git pull` in a loop** — check `git status --porcelain` first. `--autostash` is a deferral of
>    risk, not a safety net; it broke a live repo. (L-007)
>
> **Every error logged makes the next run better. That is the entire point.**
<!-- /ledger-mandatory:v2 -->

<!-- build-correctly:v1 — MUST STAY AT THE TOP. Depth lives in the skills; this is the part that must NOT depend on a skill loading. -->
> # 🏭 HOW WE BUILD — the four layers. Non-negotiable, every build.
>
> **1. PUT EVERY RULE IN THE STRONGEST TIER IT CAN LIVE IN.** Move rules *up* this ladder, never down:
> **HOOK** (cannot be violated, needs no reading) → **CLAUDE.md** (always read) → **SKILL**
> (description loaded, body on trigger) → **BRAIN DOC** (retrieved when searched).
> **The sorting question:** *could an agent violate this BEFORE having any reason to look it up?*
> Yes → hook or CLAUDE.md. No → a skill.
> 🛑 **The skill tier is an accelerator, not a guarantee.** Proven here 2026-08-06: a skill whose
> description never reaches the model is invisible, and *nothing reports it*. Anything that must be
> certain belongs above this line, not below it.
>
> **2. NAME THE SPACE BEFORE YOU BUILD THE STEP.** **LATENT** (taste, judgement, routing) → a markdown
> skill. **DETERMINISTIC** (numbers, money, dates, SQL) → a script + a cited table. Confusing the two
> causes most agent failures. 🛑 **RULE #0 — never invent a number or a formula — is the special case.**
>
> **3. WHERE THE CHECK GOES — and whether it is checking anything.**
> **① QUALITY gate BEFORE the tests** — tests cement behaviour; if the behaviour is mediocre they
> cement mediocrity. Prove it is good first. **② STATION gate** — fail at the step that MAKES the
> defect, not at the end (*jidoka*; TPS is **not** stage-gate). **③ MERGE gate** — a change that
> doesn't improve the cases doesn't ship. **④ UPGRADE gate** — re-run on every model upgrade; when
> ablation says the model no longer needs a skill, retire the skill and **keep the eval**.
> 🛑 **The three ways a gate betrays you:** it goes **blind** (keyed to a string the work itself
> changes — rename anything, re-verify every gate watching it) · you take it **green by loosening it**
> (**fix the detector, never the check**) · it was **never negative-tested** (*a gate you have never
> seen fail is not known to work* — inject a violation, watch it fail, restore).
> **Cheap before expensive:** static/regex first; model-judged only where regex cannot reach.
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [corefluxsystems-os/ADA-Compliance-OS](https://github.com/corefluxsystems-os/ADA-Compliance-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
