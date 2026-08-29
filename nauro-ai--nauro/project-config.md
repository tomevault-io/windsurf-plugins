---
trigger: always_on
description: Seeds Nauro's project store from an existing repo. Use after `nauro adopt` has run locally. On filesystem-capable surfaces, reads docs (README, manifests, ADRs, Memory-Bank) for rationale and inspects code, config, tests, lockfiles, and recent git history for evidence, then surfaces targeted probes that turn evidence into rationale. On chat surfaces, operates on pasted content against an already-adopted project.
---


# Nauro adopt skill

The agent helps the user seed Nauro with context from the current repo. Before this skill runs, the user has run `nauro adopt` from the repo root, which created the project, wired MCP across surfaces, and installed this skill into the agent's surface directory. The agent's job here is to seed the Nauro store via MCP write tools: docs supply the rationale for documented decisions, code and config and tests and manifests and recent git history supply evidence, and the user supplies the "why" via targeted probes when only evidence is present. Do not invent rationale. Record only what was actually decided, with the reasoning that supports it.

## Surface modes

The agent's behaviour depends on whether the surface can read the repo directly.

- **Filesystem-capable surfaces** (Claude Code, Cursor, Codex CLI). The agent runs Step 0 and Steps 1–11 in full. Step 0 is an optional rapid first pass that files only the decisions carrying two verifiable citations; Docs are read for rationale in Step 3; code, config, tests, manifests, and recent git history are inspected for evidence in Step 4; targeted probes in Step 6b turn evidence into rationale by asking the user.
- **Chat surfaces** (Claude.ai, Perplexity). The agent has no shell. It operates only on content the user pastes into the chat (Step 3b), and only against an already-adopted project (verified in Step 3b). Step 0 is filesystem-only and is skipped on chat surfaces, exactly as Steps 1, 2, and 4 are; the Step 6b probes are likewise unavailable, and the agent does not ask the user to paste code in lieu of running shell commands. The skill skips from Step 3b directly to Step 5.

## Step 0 — Rapid Cited Seed

Filesystem-capable surfaces only; skipped on chat surfaces exactly as Steps 1, 2, and 4 are. Step 0 is a fast first pass that files only the decisions whose rationale and rejected alternative are each a verbatim span the agent can point at by `file:line`. It reads **no new surface** — only the same Step 3 doc set (README; manifests; CONTRIBUTING / ARCHITECTURE / DESIGN / CLAUDE.md / AGENTS.md; the ADR dirs; the Memory-Bank files). Steps 1–11 still run afterward as the deep follow-up; Step 0 never replaces them or lowers their bar. A thin or empty Step 0 is a correct outcome — disciplined refusal, not a missed number.

### Step 0.1 — Draft cited cards

The agent reads the Step 3 doc set and drafts decision **cards**. A card qualifies for the batch **only** when the agent can quote **two** verifiable spans from those docs:

1. a **rationale** span — the documented "why" for the choice, and
2. a **named rejected-alternative** span — a *distinct* option the source names and sets aside. The grammatical inverse of the choice ("we did not not-do X") is **not** a rejected alternative; the span must name a real second option (e.g. "Memcached", "a monolith", "the embedded adapter"). A deferred or conditional option the source holds open ("this may become valid later", "revisit after v2") is held open, not rejected — it does not satisfy the second span.

Each span is quoted with its `file:line`. There is **no quota**: file as many cards as carry two honest spans — even one, even zero. Never pad to a number, never weaken a span to reach a count.

The agent does not compose rationale. Every character of a card's rationale is either a span quoted from a doc or text the user types. The agent never writes a "why" of its own, never paraphrases prose into a rationale the source did not state, and never infers a rejected alternative the source did not name.

Per card, the agent prepares:

- **Title** (≤60 chars) and a one-line summary (≤140 chars).
- **Rationale span (read-only)**, shown with its `file:line`. This is source text the agent surfaces; it is not an editable field.
- **Rejected-alternative span (read-only)**, the named option plus its `file:line`.
- **An empty "your why" field**, separate from the read-only spans, left blank for the user to fill or edit. The agent never pre-fills this field with the source span or with anything else — a human-supplied or human-edited "why" lives only here, never folded into the read-only span.
- **Confidence** — `high` only when the source carries a literal ADR `Status: Accepted`; otherwise `medium`. Tone, emphasis, or a confident-sounding paragraph never promote to `high`.

### Step 0.2: Pre-check and classify each card

Before presenting the batch, the agent runs Step 7 steps 1–3 for each card: call `check_decision(proposed_approach=<title plus the one-line summary>, project_id=...)` **once per card**, triage the inline headers, read in full the decisions that bear on the card, classify the operation, and prepare the complete operation-specific proposal. Annotate each card with the related decisions and assessment from the pre-pass. Doing this up front lets the batch show the exact proposed write and any overlap before confirmation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nauro-AI/nauro](https://github.com/Nauro-AI/nauro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
