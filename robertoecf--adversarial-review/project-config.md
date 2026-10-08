---
trigger: always_on
description: This plugin implements a single principle:
---

# AGENTS.md: review agent guidelines (host-agnostic)

## Plugin architecture

This plugin implements a single principle:

> **The partner reviews, never the host.**

For every review (plan, code, or prompt — all through the single `adversarial-review` skill):

- The agent reading the SKILL.md is the **host**.
- The host **must not** review the host's own work.
- The host runs `lib/call-external.sh` to delegate the heavy critique to the
  OTHER agent (the **partner**).
- The host then runs its own independent analysis and **cross-validates**
  against the partner's output, tagging findings as `[cross-validated]`,
  `[external-only]`, or `[host-only]`.

There is **no haiku courier subagent** in this version. The host (main
session) does both the external dispatch and the synthesis. Removed in the
0.5.0 refactor: the courier added complexity (model inconsistency across
files, blocking-vs-non-blocking ambiguity) without proportionate value.

## Cross-host routing

| Detected host | Partner |
|---------------|---------|
| `claude` | Claude authors (default): T1 Astra high via Pi, then `codex exec`; T2 Pi chain; T3 Gemini. Codex authors: T1 is the Claude main loop inline; wrapper adds T2/T3, never Codex |
| `codex` | Codex/user/unknown authors: T1 `claude -p`; Claude/Pi/Grok authors: T1 Codex Astra. Then T2 Pi chain (non-xAI for Grok authors), T3 Gemini |
| `grok` | Codex Astra, then Gemini |
| `pi` | Codex Astra, then Claude, direct xAI, then Gemini |
| `unknown` | Direct xAI, then Pi, then Gemini |


Detection happens at every invocation via `lib/detect-host.sh`. See
`references/host-detection.md` for the priority order and the env-leak
asymmetry that drives "Codex env first, Claude env second".

Codex- and Claude-host callers should set `ADVERSARIAL_REVIEW_AUTHOR` to `codex`,
`grok`, `claude`, `pi`, `user`, or `unknown`. Aliases `sol`, `openai`, `astra`,
`xai`, and `anthropic` are accepted case-insensitively. On Codex, omitted authors
infer `codex`; `user` and `unknown` follow the same Grok-first route. On Claude,
omitted authors infer `claude`.

## Anti-recursion contract

`lib/call-external.sh` reads `ADVERSARIAL_REVIEW_DEPTH` (default `0`). If
it sees `≥ 1`, it refuses with exit `1`. Before invoking the partner, it
sets `ADVERSARIAL_REVIEW_DEPTH=1` in the partner's env. So if a launched
partner ever re-triggers this skill, the guard fires. Never disable.

## Critique discipline (v0.8.1+)

- **Stance**: break confidence; do not validate. No credit for intent or likely follow-up.
- **Attack surface**: prioritize expensive failures (auth/trust, data loss, rollback/idempotency, races, degraded deps, migrations, observability gaps).
- **Finding bar**: material only — what fails, why vulnerable, impact, concrete fix. Prefer one strong finding over many weak ones. Clean review with zero findings is valid.
- **Over-engineering**: every plan/code review runs a simplicity counterfactual with required behavior and safety held fixed. Challenge YAGNI, false seams, duplicate ownership, and missed in-repo reuse, including canonical Effect implementations. LOC or framework taste alone is never a finding. An OE finding requires a path/line or plan-section anchor, cited evidence plus material failure/cost/risk, and a simpler alternative that preserves behavior and safety; see `references/output-standards.md`.
- **Focus**: user focus weights the pass; other material issues still report.
- **Summary**: terse ship/no-ship first (≤200 tokens), not a neutral recap.
- Details: `references/codex-lessons.md`, `references/output-standards.md`.

## Output integrity (for synthesis by the host)

Every finding in the final output MUST include:

1. **Severity**: P0 (critical) / P1 (high) / P2 (medium) / P3 (low)
2. **Evidence**: direct quote, file:line reference, or concrete scenario
3. **Impact**: user/data/security/ops consequence
4. **Recommendation**: specific, actionable fix
5. **Confidence**: high | medium | low
6. **Origin tag**: `[cross-validated]` / `[external-only]` / `[host-only]`
   (from the cross-validation step)

Do not invent files, lines, or attack chains. Mark inferences.

## Severity scale

| Level | Meaning                                                | Action                       |
|-------|--------------------------------------------------------|------------------------------|
| P0    | Critical: exploitable, data loss, security breach      | Must fix before merge/deploy |
| P1    | High: significant risk, likely failure mode            | Should fix in this iteration |
| P2    | Medium: material quality/reliability, limited blast    | Fix when convenient          |
| P3    | Low: only if still material — not style dump           | Optional improvement         |

## Fallback chain

See `references/fallback-chain.md` and the SKILL.md roster. Order is host and author aware: T1 Claude/Codex cross-review, T2 Pi chain (non-xAI for Grok authors), T3 Gemini, then a same-family clean-context subagent, then DEGRADED mode. **The DEGRADED mode emits an explicit banner** in stdout and
returns exit 2 from `call-external.sh` so the SKILL knows to surface it to
the user.

## Honesty rules

- When a review is clean, say so. Do not manufacture findings.
- When uncertain about severity, use confidence tags.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertoecf/adversarial-review](https://github.com/robertoecf/adversarial-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
