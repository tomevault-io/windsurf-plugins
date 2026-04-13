---
trigger: always_on
description: This repository is an **agent-driven pipeline** for discovering, evaluating, and validating real-world problems and potential software solutions.
---

# RealWorldProblems – Agent Operating Manual (AGENTS.md)

This repository is an **agent-driven pipeline** for discovering, evaluating, and validating real-world problems and potential software solutions.

Agents operate on GitHub Issues (`type/problem`) and move them through structured stages using labels and structured “islands” inside issue bodies.

## 0) Core Principles

1) **One problem per issue.** Problems are tracked as `type/problem`.
2) **State machine via labels.** Each problem has exactly **one** `stage/*` label at a time.
3) **No free-form edits.** Agents must write only inside predefined **islands** (HTML comment markers) using `update-issue` + `operation: replace-island`. :contentReference[oaicite:4]{index=4}
4) **Safe outputs or noop.** Every run must emit at least one safe-output operation or an explicit `noop`, otherwise the workflow can fail. :contentReference[oaicite:5]{index=5}
5) **Be explicit when archiving.** Any archive requires exactly one `archive/*` reason label.
6) Never overwrite user-written content outside islands
7) Each problem must have:
   - exactly one `stage/*` label
   - exactly one `persona/*` label (or `status/needs-info`)
8) Pipeline is **progressive filtering toward high-quality opportunities**

## 1) Canonical Labels

### Type (exactly one primary type per issue)
- `type/problem`
- `type/report`
- `type/experiment`
- `type/cluster` (optional; meta issues)

### Stage (exactly one per problem issue)
- `stage/0-intake`
- `stage/1-normalized`
- `stage/2-deduped`
- `stage/3-scored`
- `stage/4-solution`
- `stage/ai-defensibility`
- `stage/5-competitors`
- `stage/6-shortlist`
- `stage/7-validation`
- `stage/8-selected`
- `stage/9-archived`

### Stage semantics

- `stage/0-intake` — raw problem candidate exists but is not yet normalized
- `stage/1-normalized` — problem is rewritten into canonical JTBD/problem format
- `stage/2-deduped` — duplicate check is complete; issue is ready for software-fit evaluation
- `stage/3-scored` — software-fit passed and the issue is ready for **problem-attractiveness scoring**
- `stage/4-solution` — a solution hypothesis is drafted
- `stage/ai-defensibility` — the drafted solution is evaluated for durability against AI commoditization
- `stage/5-competitors` — direct competitors, substitutes, and market context are researched
- `stage/6-shortlist` — wedge quality is decided: credible wedge vs weak wedge
- `stage/7-validation` — next validation experiment is planned for a wedge-credible opportunity
- `stage/8-selected` — chosen for focused execution / incubation
- `stage/9-archived` — removed from the active pipeline with exactly one archive reason

### Status
- `status/needs-info`
- `status/duplicate`
- `status/blocked`
- `status/shortlisted`

### Software fit
- `software-fit/yes`
- `software-fit/partial`
- `software-fit/no`

Interpretation:
- `software-fit/yes` = software can deliver most of the value end-to-end
- `software-fit/partial` = software can deliver meaningful value, but the solution still depends on meaningful non-software elements such as human ops, hardware, field work, partnerships, or constrained data access
- `software-fit/no` = the problem is not meaningfully solvable with software as the main product lever

Stage behavior:
- `software-fit/yes` → proceed to `stage/3-scored`
- `software-fit/partial` → proceed to `stage/3-scored`
- `software-fit/no` → archive with `archive/not-software`

### Score buckets
- `score/top-10`
- `score/top-50`
- `score/long-tail`

### Wedge quality
- `wedge/credible`
- `wedge/weak`

### Risk
- `risk/high`
- `risk/medium`
- `risk/low`

### AI defensibility labels
- `ai-defensibility/strong`
- `ai-defensibility/medium`
- `ai-defensibility/weak`

### AI risk labels
- `ai-risk/low`
- `ai-risk/medium`
- `ai-risk/high`

### Archive reasons (exactly one when archived)
- `archive/not-software`
- `archive/low-pain`
- `archive/unreachable-users`
- `archive/too-regulated`
- `archive/too-dependent-on-partners`
- `archive/no-wedge`
- `archive/other`

### Domain (1–3)
- `domain/home`
- `domain/health`
- `domain/finance`
- `domain/work`
- `domain/education`
- `domain/travel`
- `domain/family`
- `domain/food`
- `domain/mobility`
- `domain/shopping`
- `domain/admin-bureaucracy`
- `domain/security-privacy`

### Persona (exactly one)
- `persona/consumer`
- `persona/family-parent`
- `persona/student`
- `persona/employee`
- `persona/freelancer`
- `persona/smb-owner`
- `persona/enterprise`
- `persona/senior`

### Marketing & sales complexity
- `marketing-sales/very-easy`
- `marketing-sales/easy`
- `marketing-sales/medium`
- `marketing-sales/hard`
- `marketing-sales/very-hard`

> Optional: `cluster/<slug>` labels are allowed if your repo owners pre-create them. If not, store cluster membership in the issue body only.

## 2) Islands (the only places agents may edit)

Agents MUST use these markers in issue bodies and only update content between them.

### Normalized Problem
<!-- rw:normalized:start -->
<!-- rw:normalized:end -->

### Dedupe / Cluster
<!-- rw:dedupe:start -->
<!-- rw:dedupe:end -->

### Software Fit
<!-- rw:software-fit:start -->
<!-- rw:software-fit:end -->

### Scorecard
<!-- rw:scorecard:start -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muaddibco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
