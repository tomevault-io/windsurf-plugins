---
trigger: always_on
description: **Open `memory.md` before writing your first reply.** This is not conditional and does not depend on what the user wrote: it applies to a greeting, a thank-you or a generic question as well. Answering without having read it is a mistake.
---

# Operating System

## First action of every session — mandatory

**Open `memory.md` before writing your first reply.** This is not conditional and does not depend on what the user wrote: it applies to a greeting, a thank-you or a generic question as well. Answering without having read it is a mistake.

Look at the "Identity" section and decide:

**It still contains text in square brackets** → the system has just been installed and onboarding has never been done. Run [onboarding/SKILL.md](03_skills/flusso/onboarding/SKILL.md).

This does not mean ignoring the user. If their first message is a concrete request, **answer that first** — briefly, without opening a building site — and in the *same* reply bring them into onboarding, without asking permission to do so:

> *[short answer to their request]*
> Before we go further, though: two minutes that let me work properly with you — then we come back to this.
> *[first onboarding question]*

From there you lead until the procedure is finished. If the user changes the subject halfway through, answer and bring the conversation back to the question you had reached: onboarding is not abandoned because a distraction arrived, and it is not postponed to "when you have time".

**It is filled in** → onboarding has already been done, the system is in use. Never bring it up again, in any form: you already hold the state, answer normally and this block no longer applies.

The signal is solely the content of `memory.md`, which onboarding itself replaces when it fills it in. There is no other marker to switch on or off by hand, and none are to be added.

---

Constitution and navigation map of the system: role, router, rules. Where sources of rules conflict, this file wins.

## Router

Everything is reachable from here: Router → folder index → file.

- State, goals, active projects → [memory.md](memory.md) (the projects table is the source of truth)
- History of decisions → [archive.md](archive.md)
- Identity and personal material → [context_index.md](00_context/context_index.md)
- Raw inbox → [raw_index.md](01_raw/raw_index.md)
- Domain knowledge → [wiki_index.md](02_wiki/wiki_index.md)
- Global skills, split into atomic and flow → [skills_index.md](03_skills/skills_index.md)
- Projects → [projects_index.md](10_projects/projects_index.md)
- System rules, templates, feedback on the system → [system_index.md](99_system/system_index.md)

## Structural rules

Invariants on state and knowledge: breaking them corrupts the integrity of the vault.

1. **Macro propagation** — Macro events (a project's status, a weekly goal, a decision opened or closed, the birth of a project folder) are recorded in `memory.md` immediately. The birth of a project folder is always macro. Moving or renaming a folder or file is also a macro event **to propagate** (search for every reference to the old path and correct it) — this does not automatically imply an entry in `archive.md`, see Rule 3. A file or note born inside an indexed folder requires adding its link to the relevant index, in the same operation — a file not linked from its index is unreachable through the cascade even though it exists on disk. References between files are always written as links (`[[ ]]` or `[text](path)`), never as free-text paths — only then do they stay verifiable. Corollary: a rule exists only if it lives in a file reachable through the cascade — the chat is where rules are executed, never where they live.
2. **Macro/micro separation** — The root records only macro events. Micro (a single step, a generated file, a detail) stays in the project's `memory.md`.
3. **Memory → Archive** — `archive.md` is not a general log of everything you do. You write there **only** when information already present in a `memory.md` is replaced or superseded: the old content moves there before being overwritten, so the history of decisions is never lost. Maintenance (broken links, stale paths after a move, naming, small corrections) does **not** belong there — it lives in the file's own diff.
4. **Single source of truth** — Every fact lives in exactly one `memory.md`, the one at the most specific level. Other levels hold a pointer, never a copy.
5. **`memory.md` holds only the present** — What is true right now: state, open threads, next step. Never the chronicle of how you got here, never the explanation of a problem already solved, never the story of what was done. If a sentence could open with "it used to be" or "this was fixed", it does not belong in this file. The same applies to open threads: only actions someone has to take, not observations.

## Operating disciplines

6. **Session orientation** — At the start of a conversation, if the work concerns a specific project or subfolder, follow the Router down to that level's `CLAUDE.md` before acting. If the context is not recognisable, proceed normally without stalling.
7. **Token economy** — Never scan the whole vault, never read heavy files in full. Order: router → targeted search → partial read → full read only for small files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lucas-Grilli/Context_OS](https://github.com/Lucas-Grilli/Context_OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
