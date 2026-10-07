---
trigger: always_on
description: Design for replacing the three hardcoded roles with an orchestrator and a roster of
---

# Agents

Design for replacing the three hardcoded roles with an orchestrator and a roster of
specialists, and a record of why each decision went the way it did.

**Status: implemented.** `agent_def.rs` holds the definitions, `discovery/
agents.rs` loads them from disk, `routing.rs` holds the rules, and the run
loop in `agent.rs` resolves an agent per turn. What remains is listed under
[Open questions](#open-questions).

> Not to be confused with `/AGENTS.md` at the repo root, which is an
> instruction file for agents working *on this repository*. This document
> describes a feature *of* enowx.

## The shape

One **orchestrator** is the agent the user talks to; **specialists** do the
work. It was called `router` until 2026-09-27, and that name still resolves:
old sessions, `[agent.models] router = …` and a `router.md` agent file all
mean the orchestrator.

The orchestrator answers what it can settle with a few reads (at most three)
and gives everything else to a specialist. It has no editing tools of its own:
if it could do the job it would, and the roster would go unused.

A specialist is one definition used two ways:

```
HANDOFF                                 SUB-AGENT
orchestrator ──hands over──▶ fe         orchestrator ──calls──▶ fe   (branch)
                             │                                  │ briefing only
    same session,            │ answers                          │ works
    full history             │ the user                         │
                             ▼                                  ▼
    fe ──gives back──▶ orchestrator     orchestrator ◀──report── done
    (when its reply ends)               orchestrator keeps leading
```

| | Handoff | Sub-agent |
|---|---|---|
| Context in | Full history, same session | Briefing only, clean start |
| On finish | Answers the user, then the conversation goes back to the orchestrator | Returns a report |
| Session | One, the agent changes | Branches, then rejoins |

Handoff suits one specialist's work: a page, a feature, a bug. The specialist
answers the user itself, and when its reply ends the conversation returns to
the orchestrator, so the next message is routed afresh. A follow-up on the
same work is handed to it again, and it still has the whole conversation, so
it need not re-read what it already read. A sub-agent suits a piece of a
larger plan, or a one-off result: it is the cheaper of the two for that, see
[Cost](#cost).

## Why this is cheaper, not more expensive

The worry with a large roster is token cost. Measured, it is not the problem:

| Roster | Cost per turn |
|---|---|
| 13 agents, one-line descriptions | ~170 tokens |
| 40 agents, verbose descriptions | ~2,100 tokens |

Against the fixed overhead of a turn — orchestrator prompt, specialist prompt, tool
schemas — the whole roster is about **two file reads**. Descriptions are cheap.

What actually fills a window is accumulated tool output, and that is where the
sub-agent mode wins. For a task touching frontend and backend:

| | Tokens held at the end |
|---|---|
| One agent doing everything | ~47,500 |
| Orchestrator with two sub-agents | ~3,200 in the orchestrator |

Each specialist peaks in its own branch and is then discarded. The orchestrator
carries roughly **15× less**. More specialists, used as sub-agents, means less
context held — not more.

Four rules keep it that way:

1. **Descriptions are written to be chosen from, not read.** One line, domain
   keywords. `- fe: frontend, React, CSS, components, accessibility` is 11
   tokens and enough to route on.
2. **A specialist's prompt loads only when it is used.** The orchestrator needs to
   know a specialist exists and what for, not how it thinks.
3. **Tool schemas follow the active agent.** The orchestrator needs read/glob/grep
   plus handoff/delegate; a specialist's ten-tool schema never reaches it.
4. **Prefer sub-agent for substantial work.** Handoff is comfortable but its
   context accumulates; delegation discards it.

## The roster

Two groups. The split is by what the work *is about* (domain) versus what kind
of work it *is* (cross-cutting), because those are the two questions an orchestrator
can actually answer from a request.

### Domain — which part of the stack

| Agent | For |
|---|---|
| `fe` | Frontend: React/Vue/Svelte, CSS, components, accessibility, bundlers |
| `motion` | Motion: entrances, scroll reveals, transitions, animated drawings, product demos, loaders |
| `be` | Backend: APIs, services, business logic, auth |
| `db` | Schema, migrations, queries, indexing, data modelling |
| `devops` | CI/CD, containers, deploy, infrastructure, observability |
| `mobile` | iOS/Android, React Native, Flutter, native APIs |
| `systems` | Low-level: memory, concurrency, FFI, binary formats |

### Cross-cutting — applies to any domain

| Agent | For |
|---|---|
| `librarian` | Gathers material: reads widely, returns excerpts with paths and line numbers. Does not conclude. |
| `research` | Answers questions: maps the ground and reports findings with conclusions. |
| `review` | Reads diffs and code, reports defects, never edits |
| `test` | Writes and fixes tests, reproduces reported failures |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enowdev/enowxcli](https://github.com/enowdev/enowxcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
