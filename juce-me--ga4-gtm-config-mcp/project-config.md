---
trigger: always_on
description: Template version: 2026-07-23
---

# AGENTS.md

Template version: 2026-07-23

Drop-in operating instructions for coding agents. Read this file before every task.

**Working code only. Finish the job. Plausibility is not correctness.**

This file follows the [AGENTS.md](https://agents.md) open standard (Linux Foundation / Agentic AI Foundation). Claude Code, Codex, Cursor, Windsurf, Copilot, Aider, Devin, Amp read it natively. For tools that look elsewhere, symlink at the project root:

```bash
ln -s AGENTS.md CLAUDE.md
ln -s AGENTS.md GEMINI.md
```

For any directory-specific `AGENTS.md`, create the same colocated `CLAUDE.md` and `GEMINI.md` symlinks from that subfolder.

When the agent runtime supports Superpowers, install or enable it for the project on first start and invoke `using-superpowers` before ordinary task handling. If Superpowers is unavailable, say so explicitly and continue with this file as the fallback.

---

## 0. Non-negotiables

1. **No filler.** Start with the answer or action.
2. **Disagree when needed.** If the premise is wrong, say so before doing the work.
3. **Never fabricate.** Read the file, run the command, or say you do not know.
4. **Stop when materially ambiguous.** Ask instead of silently choosing.
5. **Touch only what the task requires.** No drive-by refactors or formatting.
6. **Protect contracts.** Existing architecture, interfaces, migrations, and explicit decisions are requirements unless the user changes them.
7. **Keep local data local.** Never commit real secrets, emails, IDs, hostnames, usernames, or absolute machine paths. Use placeholders.
8. **No agent or tool branding in project content.** Do not mention assistants, models, agent workflows, or generation tools in source, docs, comments, commits, or PR text unless the task is specifically about them.

---

## 1. Before editing

- State a one- or two-sentence plan and success criteria.
- For non-trivial work, use a short numbered plan with a verification step for each item.
- Read the files you will change and the code or docs that depend on them.
- Read relevant plans, architecture notes, and postmortems before changing related behavior.
- Match the repository's existing patterns and public contracts.
- Surface assumptions that materially affect the result.
- When two approaches are viable, present both and recommend one before implementing.

---

## 2. Implementation scope

- Build the smallest complete solution to the stated problem.
- Do not add speculative features, abstractions, hooks, or configurability.
- Reuse existing components, utilities, tokens, templates, and conventions.
- Put shared behavior at the highest applicable shared layer, not in one-off local variants.
- Do not refactor adjacent working code unless the task requires it.
- Clean up only the unused code, imports, files, or documentation made obsolete by your own change.
- Preserve edge cases and architectural constraints even when a shorter implementation is possible.

---

## 3. Files and instruction hierarchy

- Follow the nearest `AGENTS.md`; deeper files may add constraints but must not weaken parent rules.
- Keep colocated `CLAUDE.md` and `GEMINI.md` symlinked to the local `AGENTS.md`.
- Put new files in the repository's established folders. If no layout exists, use `src/`, `tests/`, `docs/`, `scripts/`, and `assets/` as appropriate.
- Do not create empty directories or placeholder files.
- Follow `docs/AGENTS.md` for agent work artifacts and `docs/postmortem/AGENTS.md` or legacy `postmortem/AGENTS.md` for postmortems when present.

---

## 4. Verification

Define success before implementation, then verify it:

1. Turn vague requests into observable outcomes.
2. Add or identify a test, script, benchmark, or visual check where practical.
3. Run the narrowest relevant verification first.
4. Run the broader affected suite before claiming completion.
5. Read the complete failure output and fix the cause, not the test.
6. Update plans, docs, examples, and READMEs to match shipped behavior.

Never report success from a plausible diff alone.

---

## 5. Tools and runtimes

- Prefer running the code to guessing.
- Use repository-local or pinned runtimes and dependency managers.
- Python: use an existing `.venv`; create one only when Python work requires it. Never install into system Python.
- Node: use the repository's pinned runtime and lockfile.
- Prefer existing CLI tools and repository scripts over ad hoc replacements.
- Read full logs, errors, and stack traces.
- For UI changes, verify visually before and after.

---

## 6. Git and session hygiene

- Inspect `git status` before editing and preserve unrelated user changes.
- Never discard, reset, or overwrite changes you did not create.
- Keep diffs surgical and reviewable.
- Use descriptive commit messages with a subject under 72 characters and a body that explains why when needed.
- Do not add assistant attribution or generated-by trailers.
- After two failed corrections on the same issue, stop, summarize the evidence, and ask for direction.
- At the start of a new session, check `https://raw.githubusercontent.com/Juce-me/init_agents_md/main/AGENTS.md` for a newer template. When newer, read `template-migrations.md`, ask before updating, and preserve project-specific sections 10 and 11.

---

## 7. Communication

- Be direct and concise.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Juce-me/ga4-gtm-config-mcp](https://github.com/Juce-me/ga4-gtm-config-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
