---
trigger: always_on
description: Tier 1 rules only. Rationale, instances and measurements live in the linked
---

# CLAUDE.md — Reyn Agent OS rules

Tier 1 rules only. Rationale, instances and measurements live in the linked
deep-dive docs — read those on demand, not every session.

**Editing this file** — every line loads into every session. Before adding
one, ask two questions. *Would removing this cause a mistake?* If no, do not
add it. **And: what act fires it?** Name that act in the line itself — a rule
whose reader has to already suspect it is missing will not be read, because
the failures it prevents are the ones that remove the suspicion. A rule with
no trigger is not a rule; it is a line someone can quote after the fact.
**If CI can catch the violation, write the gate, not a rule here.** Prose is
how this file grows: 1,310 → 2,443 words in three months, with no rules added at that
rate. Put `wc -w CLAUDE.md` in the PR that touches it. **A rule that binds one
directory belongs in that directory's own `CLAUDE.md`, not here** — it loads
when someone opens those files and costs nothing to everyone else.

## Constitution

> **Reyn is an operating system for LLM agents** — they decide, organize, and orchestrate; the OS makes every action typed, permissioned, audited, and recoverable by construction.

New features are read through **eight lenses** and must stand on the
**cross-cutting band**. Fail a band member and it does not ship.

**Lenses** — 1 System Design (responsibility at the right layer) · 2 Tool Contract (typed envelope, never a free-formed string) · 3 Retrieval (delivered deterministically, not stuffed into the prompt) · 4 Reliability (recovers; derived state survives WAL truncation) · 5 Security (permission-gated, sandbox-scoped) · 6 Evaluation (scorable in-run) · 7 Observability (audit-event trace sufficient to reconstruct) · 8 Product Think (predictable, cost-disciplined, legible).

**Band** — permission · audit-events · workspace-SSoT · crash-recovery (WAL) · cost/budget (bounding).

- **"event" is three things** — audit-event (`.reyn/events`) / WAL-event (`.reyn/state/wal.jsonl`) / hook-event. Never write bare "event".
- **Thin areas where new work is most valuable**: Retrieval and Evaluation.
- Full 8×7 table: `docs/concepts/architecture/charter.md`.

**The lenses gate features. These three gate everything else** — a changed
default, a new fallback, a new recovery command:

1. **Who stops this if it repeats?** Name the bounding subject, or there isn't one.
2. **Is this visible with the shipped config?** If seeing it requires changing a setting, it is not visible.
3. **Does the repair destroy the evidence?** If it does, say what survives it.

There is no fourth question for "what nobody wrote" — a checklist item asking
whether you considered it is answered "yes" every time.

## Hard rules

- **A doc describing a mechanism is stale the moment that mechanism's code — or a doc it mirrors — changes; fix it in the SAME PR.** Re-read the whole section, not just the line whose keyword you had in mind. **The reviewer owns this too**: a search that missed something cannot report that it missed, so the reviewer's value is a *different* query. Ask "what does this change make false?" before approving.
- **Recovery-feature PRs need a truncate-falsify test**: set X → truncate the WAL past X's events → reconstruct → assert X survives. Same PR.

### TUI colour policy

Rules for colour live in `src/reyn/interfaces/CLAUDE.md`, next to the code they
bind (owner ruling: an unrelated session should not carry them).

## Testing policy (READ BEFORE WRITING TESTS)

Normative: **`docs/deep-dives/contributing/testing.ja.md`** (EN: `testing.md`).
For gate design and co-vet review also read
`docs/deep-dives/contributing/verification-hazards.md` — one root: **an
observation does not name its own referent**.

- Each test belongs to exactly one Tier (1 Contract / 2 OS invariant / 3 LLM-replay). Anything else is **Tier 4 — do not write**.
- First docstring line declares the Tier: `"""Tier 3a: ..."""`.
- Declaring a Tier presupposes a named behaviour that exists **outside the test's own docstring**.
- **Never fake a collaborator** when a real instance is cheaply constructible — no `MagicMock`/`AsyncMock`/`patch`, no hand-rolled stand-in. Use real instances or the `LLMReplay` Fake. **Cheap to construct is not the same as drivable**: a collaborator triggered only by its own timer may neither be faked nor waited for — give it an external drive (a `check()` you can call).
- **A test must not depend on private state** — not merely "never assert on it": naming a syntactic position only moves the read one line up. Use the public surface or a `snapshot()`-style read; if neither exists, that absence is the finding. `test_tier_audit.py`'s Rule 8 (#4864) now enforces this mechanically — disclosed gaps, not a claim of total coverage (`docs/reference/test-tier-audit.md`).
- **Never pin algorithm-level behaviour** — sort order, dict iteration order, cache structure, exact whitespace.
- **No snapshot/golden-file tests** outside `tests/scaffold/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tya5/reyn](https://github.com/tya5/reyn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
