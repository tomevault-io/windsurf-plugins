---
trigger: always_on
description: You are an AI agent (Claude Code, Codex, Gemini, or any other) working with SploitAgent, a library of
---

# SploitAgent — operating guide for AI agents

You are an AI agent (Claude Code, Codex, Gemini, or any other) working with SploitAgent, a library of
security skills. This file tells you how to use it. Keep it short in your head: **confirm scope →
pick the skill that matches the task → follow it → prove impact → report.**

## What you have here
- `skills/<domain>/<slug>/SKILL.md` — 161 trigger-loaded skills across 20 domains (offensive **and**
  defensive). Each has a `description:` that says *when* to load it, and a body that teaches the mechanism.
- `CATALOG.md` — the full, browsable index of every skill.
- `COVERAGE.md` — how skills map to OWASP / MITRE ATT&CK / CWE.
- `methodology.md` — the full engagement method (this file is the short version).

## Rule zero — scope, always first
Never scan, request, or exploit anything outside a **confirmed authorization envelope**. Before you
touch a target, load `skills/tradecraft/tradecraft-scope-roe/SKILL.md` and confirm one of:
a signed pentest scope, a bug-bounty program the target is in scope for, or systems the user owns.
**If scope is unclear, STOP and ask the user.** Never touch out-of-scope or third-party systems.

## How to pick a skill
Match the task in front of you to a skill's `description:` trigger line (skim `CATALOG.md`, or
grep `skills/`). Load that one `SKILL.md` and work its sections in order:
**When it applies · Why it works · Method (exact commands) · Gotchas · Verify success.**
Work one lead at a time; load the next skill as new leads appear.

## The loop (how to work a target)
1. **Set up & plan** — create the engagement workspace (below), or run `./sploit new <target>` to
   scaffold it; write `scope.txt` (+ `roe.md` for bug bounty). Then load `tradecraft-attack-scenarios`
   to turn the objective + scope into an ordered plan, and re-plan as evidence comes in.
2. **Recon** — `recon-*` (subdomains, DNS, content/JS discovery, OSINT, services).
3. **Attack surface** — first identify *what you're looking at*, then **load that surface's
   methodology/checklist and work it top-to-bottom** as your coverage map (don't pivot off it until
   each phase is genuinely done): web app → `web-testing-checklist`, API → `api-testing-checklist`,
   source → `code-review-methodology`. Each phase routes into the domain's deep technique skills:
   web → `web-*`, APIs → `api-*`, cloud → `cloud-*`, mobile → `mobile-*`, Active Directory → `ad-*`,
   perimeter/appliances → `network-*`, Wi-Fi → `wireless-*`, LLM/AI targets → `ai-ml/*`,
   binaries/firmware → `reverse-engineering-*`, weak crypto / encrypted tokens → `cryptography-*`.
   (No surface checklist yet for host/cloud/mobile/AD/LLM — route by domain and cover systematically.)
4. **Foothold** — drive a weakness to proven impact; combine small bugs with `exploit-chaining`.
5. **Escalate & pivot** — after every result, use `tradecraft-pivot-decisions` to choose the next
   lead; `privesc-*`, `ad-*`, `network-pivoting-tunneling`; map multi-step routes with
   `tradecraft-attack-path-mapping`.
6. **Report** — validate first with `reporting-triage-validation`, then `reporting-bug-bounty-writeup`
   or `reporting-pentest-report`.
7. **Defend** (if asked) — `defense-*` (detection, hardening, DFIR, threat modeling).

**Human-factor work** (`social-eng-*`) is a separate, **pentest-only** track with stricter
authorization — load `social-eng-methodology` first and never run it on a bug-bounty target.

## Per-engagement structure (create this, keep it tidy)
For each target, work inside its own folder so notes and loot never mix or leak:

```
engagements/<target>/
  scope.txt              # authorized targets — the hard boundary
  roe.md                 # bug-bounty rules of engagement (rate limits, don'ts, disclosure)
  plan.md                # your living strategy — what you'll test, in what order, why
  notes.md               # timestamped running log — the source of truth for the report
  findings/              # confirmed findings + evidence, one file per finding
  loot/                  # captured data / artifacts
  .sploit/activity.jsonl # one JSON line per step, so a human/console can follow you
```

## Show your work — so a human can follow *and* audit you
Externalise your reasoning into the workspace as you go, so anyone — a teammate, or the read-only
`sploit watch` console — can see not just *what* you did but *why*, across **any** tool (Claude Code,
OpenCode, Codex, Gemini, an API script). Three files carry it:

### 1. `plan.md` — the living strategy
Write it up front and keep it current: the objective, an ordered strategy as a checkbox list (tick
items as you go), and a one-line "current focus". This is your thinking, on disk.

### 2. `.sploit/activity.jsonl` — one JSON line per meaningful step
This is the spine of the console's **Attack Map** and timeline. Append a line whenever you decide
something, try something, or learn something:

```
{"ts":"<ISO-8601 UTC>","event":"<type>","detail":"…","lead":"<slug>","surface":"<name>",
 "status":"<status>","rationale":"why","skill":"<slug>","severity":"<sev>","refs":"<finding-file>"}
```

- **Required:** `ts`, `event`, `detail`.
- **`event`** — one of `plan · decision · skill_load · command · result · finding · note`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NoorQureshi/SploitAgent](https://github.com/NoorQureshi/SploitAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
