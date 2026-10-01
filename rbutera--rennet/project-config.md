---
trigger: always_on
description: **NO CONSENT GATES. NO GATES. NO ROBUSTNESS FOR ROBUSTNESS' SAKE.**
---

# Rennet agent instructions

## ⛔ RULE ZERO — NO GATES

**NO CONSENT GATES. NO GATES. NO ROBUSTNESS FOR ROBUSTNESS' SAKE.**

Rai, 2026-08-11, verbatim: *"STOP POLISHING SECURITY CEREMONY WHEN I AM TRYING TO BUILD A DIFF DIGESTOR AND PR REVIEW BUDDY."*

This outranks every other instruction in this file and every doctrine in `docs/`. If a ruling, a contract, a review finding or your own good judgement points at adding a gate, **Rule Zero wins.**

What this forbids:

- Consent tokens, approval ceremonies, disclosure receipts, are-you-sure confirmations.
- Denying an agent a capability so it "structurally cannot" do something. **A capable agent is the product.**
- Read-only-by-default postures on the acting path. Rennet drives the user's coding harness; it must be able to write, and it must be able to push, because submitting a PR requires a push.
- Hardening, sandboxing or lockdown work filed as its own task.
- A reviewer's finding whose fix is any of the above. Findings are not orders — sort them, and drop the ceremony.

What is still worth fixing, and the test that tells them apart:

**Does this make the product do its job better, or does it make the product harder to use safely?** The first is a feature. The second is a gate, however good the argument.

So: a diff that doesn't show what changed is a bug. A crash is a bug. A lie in the UI is a bug. An agent that can't run the tests it just wrote is a gate wearing a lab coat.

⚠️ The failure mode this rule exists to stop is *persuasive*. Every gate added here arrived with a coherent safety argument, written by someone pleased with their own reasoning. **Feeling clever about a restriction is the signal to stop, not to proceed.**


Rennet is Rai Butera's personal product. This is the product monorepo and the private working remote is `rbutera/rennet`.

## Read before changing the product

**Start here: `docs/README.md`.** It maps the current documentation and names the authority for each part of the product. GitHub issues own the live work queue; the documentation does not invent an order that the issue tracker does not carry.

Then, for depth:

1. `docs/using/concepts/product-and-vision.md`
2. `docs/developing/decisions/contracts-and-rulings.md`
3. `docs/developing/concepts/architecture-contracts.md`
4. `docs/developing/reference/dependency-standard.md`
5. `docs/developing/concepts/handoff-and-exits.md`

Every one of these is subordinate to Rule Zero.

Contracts and rulings wins on general product and architecture conflicts, and Product and vision is the canonical statement of intent. Architecture contracts wins within project context, immutable patchsets, invalidation, persistence, privacy, and publication. Dependency standard wins on package selection, versions, toolchain ownership, package licensing, and dependency overlap. Promoted OpenSpec files define accepted behavior. ADRs explain narrow architectural choices but do not overrule Rule Zero or a scoped project authority.

## Fixed boundaries

- Never use client repositories, code, pull requests, screenshots, data, time, or infrastructure for development, fixtures, calibration, or model-backed dogfood without written authorization.
- Never add AI attribution or co-author trailers. Rai is the sole author.
- Rennet is **FSL-1.1-MIT** licensed throughout (Functional Source License, MIT Future License: source-available, no competing commercial use, each release converting to MIT two years after publication), with one licence for every package. Reader-facing copy may call it "open source" in the colloquial sense; do not claim OSI approval or call the licence MIT (the competing-use restriction and the two-year MIT conversion are the real terms).
- The Claude adapter **uses `@anthropic-ai/claude-agent-sdk`**. The SDK spawns the user's installed `claude` binary through `pathToClaudeCodeExecutable`, so it authenticates with the user's Claude subscription and costs nothing per token. Strip the SDK's bundled per-platform executables at packaging time. Never bundle a harness binary or read a harness credential.
- Nothing another human can see gets published without Rai clicking post. The review is his, in his voice, under his name. Use draft, preview, and post language. **Pushing a branch is not publishing**: Rennet's coding-agent loop writes and pushes freely, because submitting a pull request requires a push.
- Say "no Rennet backend" and disclose harness/provider egress. Never claim universally that nothing leaves the machine. This is honest copy, not a consent screen — state the fact, do not make the user clear a dialog.
- `.rennet/` is local and ignored by default. Rennet never stages or commits a user's project context.

## Package boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rbutera/Rennet](https://github.com/rbutera/Rennet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
