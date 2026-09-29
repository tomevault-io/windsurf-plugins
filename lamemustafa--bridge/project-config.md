---
trigger: always_on
description: This document defines agent-level expectations and review responsibilities for this repository.
---

# AGENTS.md

This document defines agent-level expectations and review responsibilities for this repository.

## Agents and responsibilities

- **Core implementation agent**: owns Rust/Tauri and React implementation and module-level code health.
- **Security agent**: owns credential handling, endpoint validation, and data-leak prevention checks.
- **Release agent**: owns CI, packaging, changelog/release prep, branch policy,
  dependency-license inventory, and proof that license/NOTICE resources ship
  in supported installers.
- **Docs and governance agent**: owns onboarding docs, PR templates, issue
  lifecycle, contribution licensing, provenance checks, and NOTICE updates.

## Review flow

- All code changes go through a pull request.
- Every PR must include:
  - Functional summary
  - Test or reproduction command
  - Migration impact notes if changing sync behavior
  - Security impact notes for Tally/credential changes
- Each PR must link to one line in [review-checklist.md](./review-checklist.md) as completed before merge.

## Rectification expectations

- **When non-security defects are found**: open a follow-up `Bug` issue and
  include a `Rectify` PR with root-cause and regression check.
- **When a vulnerability, credential leak, or sensitive-data exposure is
  suspected**: follow [SECURITY.md](./SECURITY.md) privately. This requirement
  supersedes public issue/PR creation until coordinated disclosure is safe.
- PRs that touch existing workflows must include rollback notes and migration compatibility.
- Keep issue triage actionable:
  - assign exactly one area label (`area:tally`,
    `area:documents`, `area:infra`, or `area:security`)
  - set one bug severity label (`severity:p1` urgent / `severity:p2`
    production impact / `severity:p3` medium / `severity:p4` cleanup)
  - avoid open "wip" tasks without acceptance evidence.
- If regression was introduced by a specific PR, link it explicitly in the rectify issue and include it in the fix PR summary.
- For non-security production regressions, use a dedicated fix branch and label (`type:rectify`).

## Private knowledge hub

A **private** cross-repo knowledge repository holds material that must **not** live in this
public repo: vulnerabilities, crash triggers, competitor teardowns, pricing, market research,
and **sensitive** protocol findings. Its name, URL and local path are supplied out-of-band to
people and agents who have access, and are deliberately **not** recorded here — see the last
bullet.

**The test for "sensitive":** would publishing it let someone **harm a user or a Tally
instance**, or hand a competitor something they **could not measure themselves in an
afternoon**? If yes, it goes in the hub and only a de-fanged rule stays here. Ordinary
request/response behaviour, field semantics and protocol invariants belong in the appropriate
topic part declared by `docs/tally/TALLY_PROTOCOL_REFERENCE.md`, which remains the canonical index.

This distinction matters because source comments cite that reference by section. **A citation
into a private document is worse than no citation** — it looks auditable and is not.

- **Consult before you build.** Before implementing any flow touching Tally, GST, portal auth,
  MCA, or a competitor feature, search the hub for the topic first. Its contribution contract
  lives in the hub itself.
- **Write sensitive findings there, not here.** A vulnerability, crash trigger, sensitive
  protocol behaviour, or market/pricing fact goes in the hub; leave only a de-fanged rule here.
- **Never reference the private repo by name, URL, or filesystem path in a committed public
  file**, and never paste an entry's sensitive body into this tree. The boundary is the point,
  and it erodes one convenient pointer at a time: the repo name, the sibling path and the
  directory taxonomy are each small disclosures that compose into a map. This paragraph
  previously carried all three and contradicted the rule directly beneath it.

If the hub is not present (fresh clone, CI, or no access), skip the consult step — it is an
enhancement, never a build blocker.

## Engineering principles

These are derived from defects actually found in this repository, not from general advice.
Each principle names the failure it prevents. Where a principle and a deadline conflict, the
principle wins — every item below is here because ignoring it already cost this project weeks.

### P1. Nothing is built past the point where it has been proven against reality

The read pipeline reached ~15,500 lines, with 445 passing tests and 15 ADRs, on a code path
that had **never returned a single row from a real Tally**. The tests passed because they ran
against a simulator this repository wrote for itself — it verified that Bridge agreed with
Bridge.

- A component may not grow beyond a working end-to-end slice without live evidence.
- Fixtures must be **captured from a real instance**, never authored by hand. A hand-written
  fixture encodes an assumption and then defends it.
- A simulator is a regression tool for behaviour already observed live. It may never be the
  first or only evidence that something works.

### P2. Make illegal states unrepresentable, rather than writing rules people must remember


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lamemustafa/bridge](https://github.com/lamemustafa/bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
