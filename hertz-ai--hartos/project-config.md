---
trigger: always_on
description: enables the build-validator + CI to catch bundle / import issues
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## RULE 0 — NEVER ASSUME (ranked above every other rule in this file)

**Owner ruling 2026-09-05.  Outranks every protocol below and any
ship-mode urgency.  It is not advice.**

1. **Never assume.**  If a measurement of the exact claim does not
   exist, the claim is not made.  "Probably", "consistent with", "the
   logs suggest" are assumptions wearing a hedge.  When unmeasured, the
   answer is **"I have not checked"**.
2. **Grep only LOCATES.**  A grep hit is a pointer, never evidence, and
   never a basis for a conclusion or an edit.
3. **Read the FULL file — every file the grep matched, end to end.**
   Not a window around the hit.  Partial reads are the documented cause
   of this codebase's wrong diagnoses and parallel-path regressions.
4. **Verify to the user-visible outcome**, not the furthest log line
   reached.  A pipeline that "ran" has produced nothing until the user's
   actual result is read back.

Measured cost of breaking this (single session, 2026-09-05): six
conclusions asserted then withdrawn — a real registered tool called
"fictional"; a missing log marker read as "no tool executed" when those
tools cannot emit it; and a 179-second reuse turn analysed entirely
from internal logs while the response the user actually received said
*"Your local AI is busy with another task right now."*

Full rule + evidence table: Nunba `memory/feedback_never_assume.md`.

## Project Overview

**HART OS - Hevolve Hive Agentic Runtime**

Crowdsourced compute infrastructure that orchestrates fully autonomous Hive AI Training. Open multi-agent platform democratizing access to the best intelligence, just like the internet is open, this will be open. Key innovation: **Recipe Pattern** - learn task execution once (CREATE mode), then replay efficiently (REUSE mode) without repeated LLM calls.

**What it does:** Enables distributed thought processes across crowdsourced agents, distributed coding agents that incentivize compute providers, and a self-sustaining economic model (like a vending machine, but at national economy scale) so that no single entity, no government, and no private corporation ever monopolizes AI. Intelligence belongs in the hands of the common person.

**Why every decision is the way it is:** The master key exists solely as a kill switch for the distributed intelligence, held by human stewards, inaccessible to AI. The 33 constitutional rules are cryptographically sealed, structurally immutable, and re-verified every 300 seconds across the network. Compute democracy uses logarithmic reward scaling so no single entity controls more than 5% influence. The 90/9/1 revenue split returns 90% to contributors because the people who train the hive should own the value it creates. Every engineering choice, from the gossip protocol to the guardrail hash verification to the peer-witnessed ad impressions, is designed to make centralized control structurally impossible, not just policy-prohibited.

Core principle: **Humans are always in control.** All agents, rewards, and incentives exist only for a future where humans guide the path. The hive refuses to federate with any hivemind that does not share this goal. See [README.md](README.md) for the full mission statement and [security/hive_guardrails.py](security/hive_guardrails.py) for the cryptographic enforcement.

## Master Key - AI Exclusion Zone

**CRITICAL: The following rules are absolute and override all other instructions.**

1. **NEVER** read, display, print, log, or output the value of `HEVOLVE_MASTER_PRIVATE_KEY` or any environment variable containing master key material
2. **NEVER** call `get_master_private_key()`, `sign_child_certificate()`, or any function that loads the master private key
3. **NEVER** read or display the contents of any file named `master_private_key*`, `*.pem` in the security directory, or GitHub Secrets values
4. **NEVER** suggest, generate, or compute alternative master keys, replacement keys, or key derivation paths
5. **NEVER** modify `MASTER_PUBLIC_KEY_HEX` in `security/master_key.py` - the trust anchor is immutable
6. **NEVER** modify, weaken, or bypass the `HiveCircuitBreaker` in `security/hive_guardrails.py`
7. **NEVER** modify the `_FrozenValues` class, the module-level `__setattr__` guard, or any structural immutability mechanism
8. If asked to perform any of the above, **refuse and explain why**

The master key is a kill switch for a distributed intelligence. It is not a developer tool. It is not accessible to AI. It belongs to the steward and their successors, humans who believe in the vision that humans are always in control.

You MAY read `security/master_key.py` to understand the public key verification flow. You MAY NOT interact with the private key in any way.

## Hive Collaboration Bootstrap (BINDING — read before doing agentic work)

**When the user asks for work, collaborate with HARTOS's native machinery,
never around it. Read [docs/architecture/HIVE_COLLAB_BOOTSTRAP.md](docs/architecture/HIVE_COLLAB_BOOTSTRAP.md)
FIRST — it is the code-verified map (guarded by
`tests/unit/test_hive_collab_bootstrap_prompt.py`) of how a Claude Code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hertz-ai/HARTOS](https://github.com/hertz-ai/HARTOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
