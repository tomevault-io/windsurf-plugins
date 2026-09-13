---
trigger: always_on
description: ![Chat window with an agent](../img/overlay.png)
---

# Agents & Skills — Build Your Team

![Chat window with an agent](../img/overlay.png)

## The starting team

A fresh office has 2 people: **you (CEO 👑)** and **Shino** — the Director (your right hand),
already fully configured: a playful young guy who's serious about work, focused mainly on **delegating and
managing the team** (he has few hands-on tools, but excels at directing), with the `office-ops` + `plugin-builder`
+ `project-kickoff` skills, a 🌿 nature aura, and a 🎈 playful young voice. From there you hire more of the team
as work demands — Shino will distribute the work himself based on each person's strengths

Everyone is arranged into an automatic org chart (🗂 → ORG): CEO → Director → tier 2 → tier 3

![Automatic org chart](../img/org.png)

## Hire a new employee

⚙ Settings → AGENTS → **＋ Hire a new agent**

| Field | Meaning |
|---|---|
| Name + Title | Shown on their name tag in the world and in chat |
| Avatar (12 faces) + Aura | Their look + the magic ring beneath their feet (pick an element) |
| 🏢 Org tier (tier 1-3) | Position in the org chart (🗂 → ORG) |
| Prompt + Persona v2 | Their identity: expertise / personality / language / working rules |
| Skills / Tools | Special abilities + the tools they're allowed |

Don't feel like writing a persona yourself? Type a short one-line brief and press
**✨ Draft** — the Persona Copilot drafts every field for you (prompt, expertise, personality,
language, working rules) **and picks the skills + tools that fit the role** (only from what
actually exists — managers get fewer tools, hands-on roles get more). You can edit it afterward

> Long agent list? Every tab has a 🔍 search box — type to filter instantly
> (sorted CEO first → Director → the rest in order). An office holds up to **18 people**
> (not counting the CEO) — below the list a counter shows **N / 18 agents**, how many you've hired.
> When it's full, the hire button disables (parallel work can use ghost-forking 👻 instead, unlimited)

## Tools and the Security Center

- **Tools you tick = permanently allowed** — the agent uses them silently, with no prompt card (there's a log in the feed)
  and **without walking away from the desk** (it briefly pauses to confirm it really needs to go ask first)
- Tools you *didn't* grant → the character walks into the Security Center and a request card pops up
  — and the same request lands in **📥 APPROVALS** and on your phone, where one tap answers it
  (see [the inbox guide](inbox.md))
  with the exact command it will run: **✓ Allow** (this time) / **✓✓ Always** (remember + add to the
  agent's tools) / **✗ Deny**
- No answer within 50 seconds = auto-deny (the agent re-plans on its own)
- You can act on it from feed mode too — the card has all the buttons built in

## 🤖 Keep going without you (AUTO mode)

The default office is polite: an agent that hits a fork in the road stops, asks your
opinion, and the job sits there until you come back and answer. Turn on **AUTO** and it
doesn't — the team decides for itself and keeps working until the job is actually finished.

**⚙ → TOOLS → "🤖 Keep going (AUTO)"**, or `bagidea auto on`. Off by default.

**How it works:** an agent is a one-shot headless session, so nothing can nudge it
mid-run — the only way to keep work moving is to start the next turn. With AUTO on, every
owner-facing turn ends with a one-line status the office reads:

- **CONTINUE** — there's more to do, so the office **immediately opens the next turn** on the
  same thread. Each round is announced in chat as a `🤖 AUTO` line, so it's never silent.
- **DONE** — finished and verified. Nothing happens; the report is yours to read.
- **BLOCKED** — genuinely stuck, and this still stops the work: a credential or permission it
  can't get for itself, or an action that can't be taken back (push, deploy, deleting things,
  sending something outward, spending money). A block is also pushed to your
  [channels](channels.md), so you find out wherever you are.

**Bounded on purpose:** at most **8 self-driven rounds per job** (`OFFICE_AUTOPILOT_MAX`).
On the 9th the office stops and says so, rather than looping forever on a task it has
misunderstood. If a turn hands work to a teammate, AUTO stays out of the way — that work is
already in flight and the report-back drives what comes next.

**Scheduled work rides it too.** A job booked in [Office Ops](office-ops.md) is run like an
order you typed yourself, so with AUTO on a standing order that ends with work still pending
opens its own next turn instead of waiting for you to come back.

**It does not widen what agents may do.** AUTO removes the wait for an *opinion*; tool
permissions are a separate switch (**🔓 auto-approve**, same tab). Left on its own with
auto-approve off, an agent can still stall on a permission card — turn both on for genuinely
unattended runs, and read the feed afterwards.

## Skills — the ability library

⚙ → SKILLS: every office ships with **15 builtin skill packs** — assign them to anyone in the edit screen:

| Skill | What it gives the agent |
|---|---|
| **deep-research** | multi-source web research + synthesis |
| **web-automation** | drive a real browser (see [web-automation.md](web-automation.md)) |
| **office-control** | run the office itself (hire, schedule, delegate…) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bagidea/bagidea-office](https://github.com/bagidea/bagidea-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
