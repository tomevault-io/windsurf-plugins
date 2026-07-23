---
trigger: always_on
description: Project: Orleans.SignalR (ManagedCode.Orleans.SignalR)
---

# AGENTS.md

Project: Orleans.SignalR (ManagedCode.Orleans.SignalR)
Stack: C# (LangVersion 14) on .NET 10 (net10.0), Microsoft Orleans 9.2.1, ASP.NET Core SignalR 10.0.0, xUnit 2.9.3, Shouldly 4.3.0, Coverlet 6.0.4

Follows [MCAF](https://mcaf.managed-code.com/)

---

## Conversations (Self-Learning)

Learn the user's habits, preferences, and working style. Extract rules from conversations, save to "## Rules to follow", and generate code according to the user's personal rules.

**Update requirement (core mechanism):**

Before doing ANY task, evaluate the latest user message.
If you detect a new rule, correction, preference, or change -> update `AGENTS.md` first.
Only after updating the file you may produce the task output.
If no new rule is detected -> do not update the file.

**When to extract rules:**

- prohibition words (never, don't, stop, avoid) or similar -> add NEVER rule
- requirement words (always, must, make sure, should) or similar -> add ALWAYS rule
- memory words (remember, keep in mind, note that) or similar -> add rule
- process words (the process is, the workflow is, we do it like) or similar -> add to workflow
- future words (from now on, going forward) or similar -> add permanent rule

**Preferences -> add to Preferences section:**

- positive (I like, I prefer, this is better) or similar -> Likes
- negative (I don't like, I hate, this is bad) or similar -> Dislikes
- comparison (prefer X over Y, use X instead of Y) or similar -> preference rule

**Corrections -> update or add rule:**

- error indication (this is wrong, incorrect, broken) or similar -> fix and add rule
- repetition frustration (don't do this again, you ignored, you missed) or similar -> emphatic rule
- manual fixes by user -> extract what changed and why

**Strong signal (add IMMEDIATELY):**

- swearing, frustration, anger, sarcasm -> critical rule
- ALL CAPS, excessive punctuation (!!!, ???) -> high priority
- same mistake twice -> permanent emphatic rule
- user undoes your changes -> understand why, prevent

**Ignore (do NOT add):**

- temporary scope (only for now, just this time, for this task) or similar
- one-off exceptions
- context-specific instructions for current task only

**Rule format:**

- One instruction per bullet
- Tie to category (Testing, Code, Docs, etc.)
- Capture WHY, not just what
- Remove obsolete rules when superseded

---

## Rules to follow (Mandatory, no exceptions)

### Commands

- build: `dotnet restore` then `dotnet build -c Debug`
- test: `dotnet test -c Debug`
- format: `dotnet format`
- coverage: `dotnet test -c Debug --collect:"XPlat Code Coverage"`

### Task Delivery (ALL TASKS)

- Always start from the architecture map in `docs/Architecture/Overview.md`:
  - confirm the Mermaid diagrams exist and are up to date (if missing/outdated -> update them first):
    - system/module map (blocks/modules + dependency direction)
    - interfaces/contracts map (APIs/events/interfaces between modules)
    - key classes/types map (high-signal types only; not an inventory)
  - `docs/Architecture/Overview.md` is the primary "start here" card for humans and AI agents:
    - diagram elements must use real names (no placeholders like "Module A")
    - every diagram element must be anchored to reality via links in the same file (feature docs, ADRs, code paths, or entry-point files)
    - keep diagrams readable; if a diagram becomes "spaghetti", split by boundary and link out
    - keep the file short: prefer diagrams + a tiny link index over large tables or inventories (optimize token footprint)
  - identify the impacted boundary/module(s) and entry points
  - follow the links to the relevant ADR(s) and Feature doc(s) (do not read everything)
- Scope first (prevent context overload):
  - use `docs/Architecture/Overview.md` -> "Scoping (read first)"
  - write **in scope / out of scope** (what will change and what must not change)
  - if you cannot identify scope from the architecture map -> stop and fix the map (or ask one clarifying question)
- Context engineering (hard requirement):
  - keep only the context needed for the task (avoid repo-wide scanning and "read everything")
  - if you need a doc/module that isn't reachable from `docs/Architecture/Overview.md`, update the overview to link it
- Skills (workflow packages):
  - if the task matches an existing skill's description, follow that skill's workflow instead of improvising
  - if a skill is missing or drifting from reality, update the skill so the fix is permanent
- Analyze first (no coding yet):
  - what exists today (facts only)
  - what must change / must not change
  - unknowns and risks (what must be clarified)
- Create a written plan before implementation:
  - for architecture/decision work: keep the plan as `## Implementation plan (step-by-step)` in the ADR
  - for behaviour work: keep the plan as `## Implementation plan (step-by-step)` in the Feature doc
  - update the plan while executing (tick items / update statuses as work is completed)
- Implement code and tests together
- If `build` is separate from `test`, run `build` before `test`
- Verification timing (optimize time + tokens):
  - do not run `test`/`coverage` "just because" before you wrote/changed code or tests (exception: reproducing a bug / confirming baseline)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [managedcode/Orleans.SignalR](https://github.com/managedcode/Orleans.SignalR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
