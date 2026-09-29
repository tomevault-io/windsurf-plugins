---
trigger: always_on
description: |
---


> **ACTIVATION RULE: When this skill is loaded, ALWAYS begin your first response with "🏁 adversarial-sprint skill active" so the operator has visual confirmation.**

# Adversarial Sprint — Skill

A SKILL asset for agents (Droid, Claude Code, Codex, etc.) that
work on a project adopting the adversarial-sprint framework. This
skill teaches the agent the project's rules + WHY + when-to-invoke
the runner **without** carrying the whole rule set inline. It is
the lightweight pointer per the discussion at chunk 7 / chunk 9:
the **digest is in the skill body** (durability across long-context
sessions), while the **full rules are referenced by index**.

## Skill digest (load-bearing principles — embedded so they
survive compaction)

These are the load-bearing invariants distilled from
`tools/OPERATING-RULES.md`. They are not the full text — they are
the distilled forms that, if dropped, break a real decision the
agent makes.

1. **Every droid call is a script invocation. No manual paste.**
   §1 / §9 — the orchestrator script is the default; manual paste
   is a paragraph-not-a-script.
2. **Assert on artifacts** (file SHAs, git log entries, signed
   bundles), **never on exit codes or plausible strings.**
   §7 — silent-green is the platform's default failure mode.
3. **Prompts describe problems and constraints, not fixes.**
   §13 — the executor is a solver, not a sed-command.
4. **Droid exec routes through `tools/run-with-model.sh`;
   envelopes parse through `tools/adapters/factory.py`.**
   §14 — never raw.
5. **Git history is reality.** §15 — never judge a phase on
   uncommitted working-tree state alone.
6. **Refuse unbounded foundation programs. Name 1–3 deliverables.**
   §17 — capacity envelopes are the rule, not the goal.
7. **Compose existing primitives; fix ergonomic friction inline;
   build in chunks; review at the end; distill reusable principles.**
   §18 — *this rule* itself; if you find yourself reinventing,
   you are running afoul of the framework's own methodology.
8. **Commit when the recommendation is clear; ask only at true
   operator-value tradeoffs.** §19 — 3-option "you choose" questions
   are appropriate ONLY when the alternatives are roughly symmetric
   across operator preference and you cannot rank them. Otherwise
   ship the recommendation with a one-paragraph WHY; the
   spec-review in §18 catches wrong recommendations.
9. **Chunk close is gated, not declared.** §20 — every chunk close
   produces `chunk-N.token.json` (HMAC-SHA256 by `EVIDENCE_SIGNING_KEY`).
   The next-chunk-start path refuses without a verifiable token
   for the prior chunk. Skills loaded as documentation of intent
   are not enforcement; the chunk-close gate is. Do not interpret
   absence of the operator-eye signal as "skill exhausted" — the
   exhaust framing cannot render anything; absence is a runtime
   contract violation with the operator-eye troubleshooting
   checklist (PRD §11 Phase 5 §5).
10. **Reviewer attestations are evidence, not assertion.** §21 — every
    chunk-close token's reviewer `envelope_sha256` MUST be computed
    from a real reviewer envelope on disk (the SHA of a fired
    `droid exec` output written to
    `evidence/phase-4.5/build-evidence/<run-id>/<chunk-id>/<reviewer>.json`).
    Build-time fixture markers — homogeneous leading-character hex
    runs, typed-in placeholders, "5555...55501"-style tokens — fail
    `tools/cross_family_review.py`'s envelope-authenticity check
    (KN-A-5 / design-doc §10). The skill does not shorten this
    distance: the operator-side `chunk-N.token.json` IS the review,
    and a token without a verifier-traceable envelope is a
    self-declaration, not a verdict. Tier-3 fix: an off-process
    signing daemon (`planning/phase-4.5/DESIGN-DAEMON-SIGNER.md`) the agent
    POSTs envelope paths to; the daemon holds the key and refuses
    if any envelope is absent on disk.
11. **Author is not the verifier, even within the same model-family
    rule.** §22 — family-distinctness (§17.2) is necessary but not
    sufficient: same-session identity carries the implementer's
    working memory into the reviewer's context. The chunk-close
    signing authority is held by a separate process (Tier-3
    daemon / §21); the reviewer-firing path is held by a separate,
    persistent cross-family referee (`planning/phase-4.5/DESIGN-PERSISTENT-REFEREE.md`,
    KN-A-8). A build agent may post `REVIEW REQUEST:` lines to
    `STEER.md` (machine-local cross-agent queue per §1). The
    build agent MUST NOT fire `droid exec` against the reviewer
    model IDs the agent itself selected; MUST NOT hold
    `EVIDENCE_SIGNING_KEY`; MUST NOT write directly to
    `evidence/phase-4.5/tokens/chunk-N.token.json`. The chunk-close gate
    enforces the structural predicate (`envelope-on-disk SHA`);
    the persistent referee enforces the identity predicate
    (signing authority / reviewer-firing process). Both must
    hold. Defense-in-depth with rule #10.
12. **Validators are not each other's paraphrase; referee fires or
    audits, never both at once.** §23 — even with §17.2 family
    distinctness and §21 envelope-on-disk distinctness, a coerced
    Tier-2 validator's content can be the build agent's
    rationalization parroted back. The persistent referee checks
    content-distinctness (length-normalized Jaccard over the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Roderick-Clemente/steel-man](https://github.com/Roderick-Clemente/steel-man) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
