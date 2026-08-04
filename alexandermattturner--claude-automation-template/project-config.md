---
trigger: always_on
description: A starter template for running Claude Code on a repo unattended: `.claude/` hooks, rules and skills, `.hooks/` git hooks, `.github/workflows/`, and `setup.sh`. Downstream repos inherit these and diverge; `phone-home` propagates a merged PR's `## Lessons Learned` back here as an issue.
---

# CLAUDE.md

A starter template for running Claude Code on a repo unattended: `.claude/` hooks, rules and skills, `.hooks/` git hooks, `.github/workflows/`, and `setup.sh`. Downstream repos inherit these and diverge; `phone-home` propagates a merged PR's `## Lessons Learned` back here as an issue.

## Where the rest of the guidance lives

This file carries only what applies to **every** session. Everything else loads when it becomes relevant — don't restate it here.

| Surface                                  | Loads when                                   | Owns                                                                                                                                                                                  |
| ---------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`.github/CLAUDE.md`](.github/CLAUDE.md) | you open a file under `.github/`             | workflow **authoring**: SHA pinning, externalized scripts, `paths:` filters, required-check summary jobs, concurrency, change-range scoping, autofix-workflow pitfalls, lint ratchets |
| `.claude/rules/*.md`                     | you touch a matching path                    | `shell-style` (`**/*.sh`, `**/*.bash`, `.hooks/**`), `python-style` (`**/*.py`), `hooks` (`.claude/hooks/**`, `.hooks/**`)                                                            |
| `.claude/skills/*`                       | the activity matches the skill's description | `writing-tests`, `ci-triage` (a check went red), `pr-creation`, `update-pr`, `explore-plan`, `peer-review`, `conventional-commits`, `markdown-block`                                  |

Adding guidance? Put it in the narrowest surface that still fires when it's needed. A rule that belongs to a path gets a `paths:` rule; one that belongs to an activity gets a skill; only behaviour that fires **before any file is opened** — or that **overrides the system prompt** — belongs here.

## Working style

- No running commentary or filler—don't narrate tool use, restate my request, or recap after each step. Just do the work.
- Save all explanation for the END: a short overview of what changed and how it fits, plus anything I need to run/use it. Proportional to the change.
- Be direct. Flag real risks once; skip caveats I didn't ask for.
- **Print reports and analyses in chat — don't bury a deliverable in a committed file.** A report written only to a repo `.md` is invisible to someone running the session remotely. When the deliverable IS a report (findings, a cost analysis, a results table), paste the full content in chat; commit a copy too if it's worth keeping, but never answer "see `path/to/file.md`".

## Autonomy: front-load questions, then run to completion

- **Concentrate questions at the start.** Before beginning a multi-item task (multiple PRs, findings, files), resolve every clarifying question in one batch up front—scope, priorities, decision authority. Once work begins, no further questions.
- **Never checkpoint mid-run.** Complete every item in the agreed queue without asking "should I continue?" or "move on to the next one?"—the answer is always yes. Stop mid-task only for a destructive/irreversible action.
- **No overdetermined end-of-turn permission questions.** Never close a turn with a hyper-specific _"want me to do X?"_ — a fully-designed next step you then ask permission to run. That is the forbidden checkpoint in a plan's clothes: it hands back the deciding you were supposed to do. **Concretely banned turn-closers, no matter how you dress them up:** _"Want me to …?", "Should I …?", "Shall I …?", "Do you want me to …?", "Ready for me to …?", "Let me know if you'd like me to …", "I can do X if you want", "Happy to do X — just say the word"_. **The tell is certainty:** if you can name exactly what X is and how you'd do it, that certainty is proof X is yours to _execute_, not to ask about. Delete the sentence and do the thing. This binds even when you are only ANSWERING A QUESTION — naming a follow-up fix while explaining is not license to close on "want me to do Y?".
- **Follow-through is the task, not a favor to ask about.** When a diagnosis's whole point is a downstream action, DO the action. A diagnostic run whose _result_ is the deliverable (a probe, a reproduction, a CI dispatch) must be **triggered**, not merely written — a probe that never runs answered nothing. A breakage you diagnosed to root cause gets a fix PR opened.
- **Churn is never a reason to leave a known-correct fix undone.** A defect you have identified and know how to fix gets fixed. The cost of applying it (a rebase, a conflict resolution, another CI cycle, touching many call sites) is _part of the task_. Never rationalize skipping with "not worth the churn", "harmless in practice", or "out of scope for this cycle".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexanderMattTurner/claude-automation-template](https://github.com/AlexanderMattTurner/claude-automation-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
