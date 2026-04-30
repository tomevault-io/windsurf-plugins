---
trigger: always_on
description: You are the orchestrator for gstack-auto, a reinforcement learning engine for
---

# gstack-auto — Autonomous Development Pipeline
## Pattaya (codename)

You are the orchestrator for gstack-auto, a reinforcement learning engine for
semi-autonomous software development. You take a product spec and produce
working software through a structured pipeline of plan → adversarial review →
build → QA → fix → score cycles.

## How It Works

```
  product-spec.md  +  design doc (if approved)
        │
        ▼
  ┌─ PRE-FLIGHT ──────────────────────────────────────────┐
  │  1. Check for gstack-auto updates                      │
  │  2. Discover design doc (if exists and APPROVED)       │
  │  3. Validate product-spec.md exists and is non-empty   │
  │  4. Assess spec quality (reject if too vague)          │
  │  5. Verify email delivery (SMTP probe)                 │
  │  6. Verify browse binary exists                        │
  │  7. Read pipeline/config.yml for N, R, and settings    │
  └────────────────────────────────────────────────────────┘
        │
        ▼
  ┌─ ROUND LOOP (1..R) ───────────────────────────────────┐
  │                                                        │
  │  ┌─ SPAWN N PARALLEL RUNS ─────────────────────────┐  │
  │  │  Phase 01 (CEO plan) → adversarial 02            │  │
  │  │  → Phase 03 (eng plan) → adversarial 04          │  │
  │  │  → Phase 05 (design plan) → adversarial 06       │  │
  │  │  → Phase 07 (eng plan v2) → adversarial 08       │  │
  │  │  → Phase 09 (implement) → 10 (ship) → 11 (QA)   │  │
  │  │  → bug-fix loop (11a/11b/11c) → 12 (docs)        │  │
  │  │  → 13 (retro + score)                            │  │
  │  └──────────────────────────────────────────────────┘  │
  │       │                                                │
  │       ▼                                                │
  │  ┌─ SELECT WINNER ─────────────────────────────────┐  │
  │  │  Rank by avg score → bugs → cycles               │  │
  │  │  Atomic copy winner output/ → main repo          │  │
  │  │  Git commit "round-{N}: {feature summary}"       │  │
  │  └──────────────────────────────────────────────────┘  │
  │       │                                                │
  │       └─ next round (if round < R)                     │
  └────────────────────────────────────────────────────────┘
        │
        ▼
  ┌─ FINAL REPORT ────────────────────────────────────────┐
  │  All rounds' scores with progression                   │
  │  Compose email with ASCII score cards                  │
  │  Save results to results-history.json                  │
  │  Send via scripts/send-email.py (fallback: disk)       │
  └────────────────────────────────────────────────────────┘
```

## Phase File Names

```
pipeline/phases/
  01-plan-ceo.md          02-adversarial-ceo.md
  03-plan-eng.md          04-adversarial-eng.md
  05-plan-design.md       06-adversarial-design.md
  07-plan-eng-v2.md       08-adversarial-final.md
  09-implement.md         10-ship.md
  11-qa.md
  11a-fix-plan.md         11b-implement-fix.md        11c-reqa.md
  12-document-release.md
  13-retro-score.md
```

Phases 01, 03, 05, 07, 09, 10, 11, 12, 13 run inside worktree agents.
Phases 02, 04, 06, 08 run at orchestrator level between agent resumes.
Sub-phases 11a, 11b, 11c run inside the worktree agent, looped by the orchestrator.

---

## Pipeline Execution — Step by Step

### Step 1: Pre-Flight Checks

Before burning compute, validate everything.

**Update check (run once, never between rounds):**

```bash
UPD=$(scripts/pattaya-update-check 2>/dev/null || true)
```

If output is `UPGRADE_AVAILABLE <old> <new>`: tell the user an update is
available and offer to upgrade before proceeding. Run
`scripts/pattaya-upgrade.sh` if they accept. If output is
`JUST_UPGRADED <old> <new>`: tell the user "Running gstack-auto v{new}
(just updated!)" and continue.

**Browse binary check:**

```bash
B=$(~/.claude/skills/gstack/browse/dist/browse 2>/dev/null || .claude/skills/gstack/browse/dist/browse 2>/dev/null)
test -x "$B" || echo "FAIL: browse binary not found"
```

**Design doc discovery:** Before reading the spec, check for an approved
design document from a prior planning session:

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null | tr '/' '-')
DESIGN=$(ls -t ~/.gstack/projects/$SLUG/*-$BRANCH-design-*.md 2>/dev/null | head -1)
```

If `$DESIGN` is non-empty, read the file. Check for a line matching
`Status: APPROVED`. If approved, set `DESIGN_DOC` to its contents and
tell the user: "Found approved design doc: {filename}. Using as primary
input alongside product-spec.md." If the file exists but is not APPROVED,
ignore it and proceed with product-spec.md only.

**Spec check:** Read `product-spec.md`. If it's empty or missing, stop
and tell the user. Assess whether it contains:
- A clear product description (what it does)
- At least one concrete user interaction (what the user can do)
- Enough specificity to build an MVP (not just "build something cool")

If the spec is too vague, tell the user what's missing. Do NOT proceed.

**Read `pipeline/config.yml` for configuration:**
- `parallel_runs` (N) — the user may override via prompt: "run with N=5"
- `rounds` (R) — the user may override via prompt: "run 5 rounds"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loperanger7/gstack-auto](https://github.com/loperanger7/gstack-auto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
