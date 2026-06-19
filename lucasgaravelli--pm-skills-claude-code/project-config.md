---
trigger: always_on
description: **27 skills** organized in **6 domains** covering the full PM lifecycle: from vague idea to scaled product. Includes complete methodological tutorials for the **0→1** journey (idea to PMF) and **1→100** journey (PMF to scale).
---

# PM Skills for Claude Code — 27 Product Management Skills

**27 skills** organized in **6 domains** covering the full PM lifecycle: from vague idea to scaled product. Includes complete methodological tutorials for the **0→1** journey (idea to PMF) and **1→100** journey (PMF to scale).

Frameworks: Teresa Torres (OST), Ash Maurya (Lean Canvas), Sean Ellis (PMF), Rob Fitzpatrick (Mom Test), Alberto Savoia (Pretotyping), Marty Cagan (Dual-Track), Lenny Rachitsky (Growth + 50 leaders).

---

## Installation

### Option 1: Clone (recommended)
```bash
git clone https://github.com/flowgrammers/pm-skills-claude-code.git
cd pm-skills-claude-code
# Open Claude Code here — all 27 /commands are available
```

### Option 2: Global install (available in any project)
```bash
# Unix/macOS/WSL
./scripts/install-global.sh

# Windows PowerShell
.\scripts\install-global.ps1
```

### Option 3: Manual
Copy files from `.claude/commands/` to `~/.claude/commands/`

---

## Journey 0→1: From Idea to Product-Market Fit

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   PHASE 1    │    │   PHASE 2    │    │   PHASE 3    │    │   PHASE 4    │
│  Problem     │───▶│  Problem     │───▶│  Solution    │───▶│  Solution    │
│  Discovery   │    │  Validation  │    │  Discovery   │    │  Validation  │
│              │    │              │    │              │    │              │
│ /persona     │    │ /lean-canvas │    │ /opp-tree    │    │ /experiment  │
│ /discovery   │    │ /competitive │    │ /hypothesis  │    │ /ab-test     │
│ /journey     │    │ /hypothesis  │    │ /experiment  │    │ /hypothesis  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
        │                                                           │
        ▼                                                           ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   PHASE 5    │    │   PHASE 6    │    │   PHASE 7    │
│  MVP Spec    │───▶│  Launch &    │───▶│  Product-    │
│  & Build     │    │  Traction    │    │  Market Fit  │
│              │    │              │    │              │
│ /prd         │    │ /launch-check│    │ /measure-pmf │
│ /user-stories│    │ /gtm         │    │ /interview   │
│ /pre-mortem  │    │ /north-star  │    │ /stakeholder │
└──────────────┘    └──────────────┘    └──────────────┘
```

**Full tutorial**: [tutorial/01-zero-a-um.md](tutorial/01-zero-a-um.md)

---

## Journey 1→100: From PMF to Scale

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   PHASE 8    │    │   PHASE 9    │    │   PHASE 10   │    │   PHASE 11   │
│  Growth      │───▶│  Scaling &   │───▶│  Optimization│───▶│  Maturity &  │
│  Engine      │    │  Expansion   │    │  & Moat      │    │  Reinvention │
│              │    │              │    │              │    │              │
│ /north-star  │    │ /pricing     │    │ /strategy    │    │ /strategy    │
│ /okr         │    │ /icp         │    │ /competitive │    │ /discovery   │
│ /experiment  │    │ /battlecard  │    │ /okr         │    │ /lean-canvas │
│ /ab-test     │    │ /roadmap     │    │ /stakeholder │    │ /prioritize  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

**Full tutorial**: [tutorial/02-um-a-cem.md](tutorial/02-um-a-cem.md)

---

## 27 Skills by Domain

### DISCOVERY (7 skills) — Understand the problem and market

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/persona` | Generates personas with JTBD, behaviors, pain points | Project start, before any spec |
| `/discovery` | Structures discovery cycle (framing → hypotheses → experiments) | Ill-defined problem, multiple hypotheses |
| `/interview-synthesis` | Synthesizes interviews into JTBD, patterns, recommendations | After interview rounds (3+) |
| `/competitive-analysis` | Maps strengths, weaknesses, differentiation gaps | Preparing PRD or strategy |
| `/opportunity-tree` | Teresa Torres OST (outcome → opportunities → solutions) | Deciding where to invest in discovery |
| `/hypothesis` | Structures testable hypotheses with metrics and kill criteria | Before any experiment |
| `/customer-journey` | Journey map (7 stages + emotions + pain points) | Understanding end-to-end experience |

### DELIVERY (3 skills) — Specify the solution

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/prd` | Complete PRD in 8 sections | Validated hypothesis, time to spec |
| `/user-stories` | INVEST user stories with acceptance criteria | PRD approved, break down for eng |
| `/acceptance-criteria` | Given/When/Then covering happy path + edge cases | Story approved, detail for QA |

### STRATEGY (7 skills) — Prioritize, plan, align

| Command | What it does | When to use |
|---------|-------------|-------------|
| `/prioritize` | RICE scoring + trade-offs + recommendation | 5+ features, need to sequence |
| `/strategy` | Strategy canvas (9 sections) | New quarter, direction change |
| `/roadmap` | Quarterly roadmap with phases and dependencies | Communicate plan to team/board |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucasgaravelli/pm-skills-claude-code](https://github.com/lucasgaravelli/pm-skills-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
