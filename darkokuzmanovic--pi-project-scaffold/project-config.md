---
trigger: always_on
description: Use when scaffolding, backfilling, or upgrading a `.pi/project/` template. Triggers on 'scaffold pi-project', 'init pi-project', 'set up .pi/project', legacy '.pi-project' wording, and natural fresh-project prompts like 'create me a new project', 'scaffold an app', 'plan my new CLI', or 'grill me on a new project'. Classifies risk as A temporary/learning, B architecture risk, C too much scope, or D not sure.
---


# Pi-Project Scaffold

Interview the user about a new project, identify the main way the project could go wrong, then write the risk-appropriate subset of the `.pi/project/` v0.4.1 template in one batch. The user supplies thought; you handle file mechanics. **Right-sizing is the point** — small projects should NOT get the full scaffold. See `## Risk classification` below.

**This skill is self-contained.** Do not invoke brainstorming or writing-plans as a prerequisite. The interview below replaces them — it covers vision, scope, constraints, milestones, and decisions in one flow.

Templates live in `templates/` next to this file. Each `.tmpl` file uses HTML comments `<!-- PLACEHOLDER: description -->` to mark substitution points. Pure-literal files (VERSION, gotchas-link.md, watch.md, polish.md) are copied verbatim.

## When to use

**Explicit triggers** (always engage): user says one of the literal trigger phrases in the description above.

**Implicit triggers** (engage by default, but acknowledge): user asks to *create a project and write a plan*, *scaffold a new app/tool*, or *grill them on a new project*. CWD has no `.pi/project/` folder. The user's request is forward-looking project work, not a one-shot script. In this case, open with: "Before I write a plan, I want to use the `pi-project-scaffold` skill — it'll grill you on vision, scope, and milestones, then write a `.pi/project/` charter you can hand off to a cheap executor model. Sound right?" Wait for confirmation, then proceed with the interview. If the user declines, fall back to writing a plain `PLAN.md`.

**Past misses to learn from:** the `claude-export` project (May 2026) was a textbook implicit-trigger case ("Create me ~/code/claude-export and write me a plan for the app. Grill me.") but the skill didn't engage; the executor wrote ad-hoc uppercase `CHARTER.md`/`PLAN.md` without the scaffold's discipline. The user re-derived the milestone-oracle pattern manually. Engage on implicit triggers to avoid this miss.
(See triggers above.) Required precondition for either trigger path: CWD has **no** `.pi/project/` folder AND no legacy `.pi-project/` folder. If legacy `.pi-project/` exists, refuse with the migration command (see `## When NOT to use`).

## When NOT to use

- Project already has `.pi/project/` — refuse and explain. Never overwrite.
- Project has legacy `.pi-project/` (from v0.3.x or earlier) — **refuse and print the migration command**, do not scaffold alongside, do not auto-migrate. Use this verbatim:
  > "Legacy `.pi-project/` detected. v0.4.1 writes to `.pi/project/`. Migrate first, then re-invoke this skill:\n>\n>     mkdir -p .pi && git mv .pi-project .pi/project && echo v0.4.1 > .pi/project/VERSION\n>     # then ensure `.gitignore` has `/.pi/*` and `!/.pi/project/`\n>\n> Re-run the trigger after the move completes."

  Do NOT execute the migration yourself — the user owns destructive git operations on their own project.
- One-shot scripts (<1 session of work) — recommend skipping the template entirely.
- User just wants to discuss the methodology, not scaffold. Point to `Dev/Projects/pi-project/template.md` in their Obsidian vault.

## Risk classification

Before the interview, ask the user what kind of problem the scaffold should protect against. In plain language: **what is most likely to make this project go wrong?** The answer determines which `.pi/project/` files get scaffolded — small projects should NOT get the full template.

**Ask exactly this question (no preamble, no narration):**

> **What is most likely to make this project go wrong?**
>
> - **(A) Temporary / learning project** — quick experiment, one-off script, or code that probably won't matter next week
> - **(B) Architecture risk** — hard bugs may come from background processes, async/concurrency, lifecycle hooks, global state, security, or integration with systems/code we don't own
> - **(C) Too much scope** — the idea is clear, but we may keep adding features and make it too big before shipping
> - **(D) Not sure yet** — unclear how big or complex this project will become

Use `ask_user` with these four options. Do NOT auto-decide. Do NOT collapse this into a yes/no — the four-way answer is load-bearing. If the user is confused, explain: “risk” means “the main way this project could go wrong,” then recommend an option from their project description and ask them to confirm.

### Route by answer

**(A) Temporary / learning project — STOP and exit the skill.**

Respond verbatim: *"Pi-project-scaffold is overkill for a temporary or learning-only project. Suggest `git init`, write code, ship it. I can write a one-paragraph README at the end if useful. Want me to exit this skill?"* Wait for confirmation. If user says yes, exit. If user says "actually I want the scaffold anyway," treat as (D).

**(B) Architecture risk — full scaffold.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DarkoKuzmanovic/pi-project-scaffold](https://github.com/DarkoKuzmanovic/pi-project-scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
