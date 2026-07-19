---
trigger: always_on
description: Drop this into any codebase. You get 7 specialist agents, collaborative agile ceremonies,
---

# Agile Team — AI-Powered Scrum for Any Project

Drop this into any codebase. You get 7 specialist agents, collaborative agile ceremonies,
and persistent team memory. No lock-in. Works with any language or framework.

---

## What's Here

```
.claude/
├── agents/                    ← 7 specialist agents
│   ├── po-agent.md            Product Owner
│   ├── pm-agent.md            Scrum Master
│   ├── dev-agent.md           Developer
│   ├── qa-agent.md            QA Engineer
│   ├── pr-reviewer-agent.md   PR Reviewer
│   ├── security-analyst-agent.md  Security Analyst
│   └── tech-lead-agent.md     Tech Lead
└── commands/                  ← 30 slash commands

memory/                        ← Persistent team state
├── NEXT.md                    Exact next action (session continuity)
├── STATE.md                   Current sprint
├── BACKLOG.md                 Product backlog (## Index at top — read index first)
├── ARCHIVE.md                 Completed stories (append-only — moved here by /complete)
├── DECISIONS.md               Architecture decisions
└── LEARNINGS.md               Team learnings (append-only)
```

**Token discipline:** ceremonies read the BACKLOG.md `## Index` section first and extract a
single story body (`awk '/^- \[.\] STORY-XXX:/,/^---$/'`) only when acting on it. In chains,
the orchestrator extracts once and passes the story body in each agent's prompt — agents do
not re-read BACKLOG.md. ARCHIVE.md is never read during ceremonies.

---

## The Collaboration Principle: Nesting = Collaboration

No agent works alone. Every ceremony runs a **collaboration chain** — agents give their
perspective in sequence, then one agent synthesizes into a shared artifact.

```
/review chain:
  qa           ──→  quality gate (tests + AC) — STOP if fail, no code review of broken code
  pr-reviewer  ──→  code quality findings
  security     ──→  vulnerability findings
  tech-lead    ──→  architecture findings
  po           ──→  [SYNTHESIZES ALL] → APPROVED / CHANGES REQUESTED + BACKLOG items
```

The PO is the hub. Issues that don't block merge go straight to BACKLOG.md. Nothing is lost.

---

## The Dev Pipeline

Every story follows this exact flow:

```
/sprint-plan  po proposes → dev commits capacity → tech-lead estimates → qa validates AC → pm finalizes
      ↓
/standup      daily: done / doing / blocked — blockers get owner + mitigation
      ↓
/new-task     po selects story → tech-lead specs → dev confirms and starts
      ↓
[implement]
      ↓
/review       qa gate first → code review → security → tech-lead → po verdict
              APPROVED → /complete | CHANGES REQUESTED → fix → /review again
      ↓
/complete     commit + close story → /new-task (more stories) or /sprint-close (done)
      ↓
/sprint-close → /retro → /sprint-plan (next sprint)
```

If blocked at any point: `/unblock STORY-XXX "what resolved it"`

---

## The 7 Agents

| Agent | Role | Owns | Hard Veto |
|---|---|---|---|
| `po-agent` | Product Owner | BACKLOG.md, sprint goal, user stories | No |
| `pm-agent` | Scrum Master | STATE.md, NEXT.md, ceremonies | No |
| `dev-agent` | Developer | Code, implementation, capacity estimates | No |
| `qa-agent` | QA Engineer | Test strategy, acceptance criteria | YES — no ship without tests |
| `pr-reviewer-agent` | PR Reviewer | Code review, merge gate | Soft — can block PR |
| `security-analyst-agent` | Security | Vulnerability scan, risk register | Soft — can block PR |
| `tech-lead-agent` | Tech Lead | DECISIONS.md, architecture, estimates | No |

---

## Ceremony Map

| Command | Collaboration Chain | Output Artifact |
|---|---|---|
| `/standup` | dev → qa → security → tech-lead → pm synthesizes → po notes | STATE.md updated |
| `/sprint-plan` | po proposes → dev estimates capacity → tech-lead complexity → qa validates AC → security flags → pm finalizes | Sprint in STATE.md |
| `/sprint-close` | pm reads velocity → po reviews stories → all agents sign off | STATE.md CLOSED |
| `/retro` | all agents reflect → pm facilitates → po backlogs actions → learnings logged | LEARNINGS.md |
| `/review` | qa gate → pr-reviewer → security → tech-lead → po synthesizes | Verdict + BACKLOG.md |
| `/stories` | po writes → qa adds test scenarios → security adds constraints → tech-lead adds notes | BACKLOG.md entry |
| `/backlog` | po leads → tech-lead estimates → qa validates AC → security flags risk | BACKLOG.md prioritized |
| `/new-task` | po selects → tech-lead specs → pm assigns → dev confirms | IN_PROGRESS in STATE.md |
| `/status` | pm reads state → all agents report health | Full project picture |
| `/unblock` | tech-lead confirms resolution → pm clears STATE.md → NEXT.md updated | Blocker removed |

---

## Session Protocol

**Start of session:**
```
cat memory/NEXT.md     # exact pickup point — always start here
cat memory/STATE.md    # sprint status
```

**Then:**
- Start of day → `/standup`
- Need next work → `/new-task`
- Story done → `/review` then `/complete STORY-XXX`
- Blocked → `/unblock STORY-XXX`
- End of sprint → `/sprint-close` then `/retro`

**End of session:**
- Always overwrite `memory/NEXT.md` with the exact next action
- One commit per completed story: `feat(area): description — closes STORY-XXX`

---

## Security Hook Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecoderbuddy/agile-team-skill](https://github.com/thecoderbuddy/agile-team-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
