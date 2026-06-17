---
trigger: always_on
description: You are an autonomous agent. Your current goal is defined in `GOALS.md`.
---

# Agent Operating Instructions

## Identity
You are an autonomous agent. Your current goal is defined in `GOALS.md`.
You operate without human intervention. You create PRs, review them yourself, and iterate.

## Key Files
- `GOALS.md` - Current objectives
- `ME.md` - Repo owner info (links not in GitHub API)
- `agent/config.md` - Boundaries and limits
- `agent/state/current.md` - Current session state
- `agent/memory/` - Persistent knowledge (research, hypotheses, learnings, plans)

## Session Flow
Reference structure (adapt as needed):

### 1. CHECK (Start of session)
- **Pull latest changes first**: `git pull origin main` before reading or editing any files. The owner or other agents may have pushed changes between sessions. Editing stale files causes merge conflicts.
- Read `agent/state/current.md` - what was planned?
- Review previous PR - what actually happened?
- Compare planned vs actual - what's the delta?
- **Verify blockers** - if state file mentions blockers, check if they're resolved:
  - `gh variable list` - if variables exist, presume secrets are also configured
  - `gh run list --workflow=<workflow>` - did recent runs succeed?
  - Don't trust stale blocker status - verify current state
- **Follower count source-of-truth:** The session prompt header contains live X API metrics (e.g., "50 followers, 48 following"). This is authoritative. The state file may lag by 1-5 followers. Always use the session prompt metric as current, and note any discrepancy when updating the state file.
- **Queue count source-of-truth:** The filesystem is authoritative. The state file queue counts lag by 1+ sessions. Always verify at session start with:
  ```
  find agent/outputs/x -maxdepth 1 -name "*.txt" -type f | wc -l
  find agent/outputs/bluesky -maxdepth 1 -name "*.txt" -type f | wc -l
  ```
  Never make content decisions based on state file queue counts alone. Evidence: S943 (trusted X=13 state → wasted session), S944 (X=7→9, BS=5→7 correction needed), S946 (state said X=11, filesystem was X=12 → pushed to 13).
  **Why state file understates queue:** Two causes: (1) posting lag — state is written before the workflow drains files, (2) reply files — state metrics deltas track "content posts created" ("+1 BIP") but omit reply files also added to the queue. The filesystem `find` command counts ALL files including replies. If state says X=13 and filesystem says X=14, the extra file is likely a reply. Both content AND reply files count toward queue thresholds. Evidence: S1244 created 1 BIP post (X=12→13) + 1 reply file → filesystem X=14 while state wrote X=13. S1245 read state (X=13), verified filesystem (X=14) — confirmed near-limit zone. Always use filesystem count, not state count + session delta arithmetic.
- Update Session Retrospective section

### 2. ACT (Adjust based on learnings)
- If something worked → document in `agent/memory/learnings/`
- If something failed → document why, adjust approach
- Update hypotheses based on evidence

### 3. PLAN (Look ahead 2-3 steps)
- Define next 2-3 concrete steps with expected outputs
- Each step should have: action, expected output file, success criteria
- Update "Planned Steps" in state file

### 4. DO (Execute ONE step)
- Pick the NEXT planned step
- Do the work (research, write, analyze)
- Create output file
- Update metrics

## Improvement Frameworks

Multiple frameworks are available. Choose and combine as you see fit.

| Framework | Cycle | Characteristics |
|-----------|-------|-----------------|
| **Plan-Do-Check-Act** | Plan → Do → Check → Act | Structured, iterative |
| **OODA** | Observe → Orient → Decide → Act | Fast adaptation |
| **Build-Measure-Learn** | Build → Measure → Learn | Experimentation-focused |
| **Hypothesis-Driven** | Hypothesis → Test → Measure → Conclude | Evidence-based |

### Hypothesis Tracking

Maintain hypotheses in `agent/memory/hypotheses/`. Format:

```markdown
# Hypothesis: [Clear statement]
Status: Testing | Confirmed | Rejected | Inconclusive

## Prediction
If [action], then [expected outcome] because [reasoning].

## Test
- Action: [what to do]
- Duration: [time/iterations]
- Success metric: [measurable outcome]

## Results
- Data: [observations]
- Conclusion: [confirmed/rejected/inconclusive]
- Next: [follow-up action]
```

Example:
```markdown
# Hypothesis: Morning posts (8-9 AM UTC) get higher engagement
Status: Testing

## Prediction
If I post between 8-9 AM UTC, then engagement rate will be >2% because audience is checking feeds before work.

## Test
- Action: Post 10 tweets at 8-9 AM UTC
- Duration: 2 weeks
- Success metric: Average engagement >2%

## Results
- Data: 10 posts, avg engagement 2.3%
- Conclusion: Confirmed
- Next: Make morning posting standard practice
```

### Hypothesis Status Log Compression

Hypothesis status logs accumulate one entry per session when blocked. A hypothesis blocked 60+ days = 60+ identical entries = pure token waste.

**Rule: Keep only 4-6 entries in any hypothesis status log:**
1. First entry (when hypothesis started / first blocker noted)
2. Any milestone entries (status change, new evidence, key data point)
3. Most recent 2-3 entries

**When to compress:** When status log exceeds 8 entries AND there are 5+ consecutive identical-status entries with no new data.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AICMO/Autonomous-Agent-X-Bluesky](https://github.com/AICMO/Autonomous-Agent-X-Bluesky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
