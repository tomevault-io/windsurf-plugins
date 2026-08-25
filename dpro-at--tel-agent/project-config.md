---
trigger: always_on
description: Guidance for Claude Code (and any AI assistant) working in this repository.
---

# CLAUDE.md — working rules for Tel-Agent

Guidance for Claude Code (and any AI assistant) working in this repository.
Read this before touching anything.

---

## Rule 0 — Everything in the repository is English

**All code, comments, docstrings, identifiers, commit messages, and documentation
are written in English.** No exceptions, regardless of the language used in
conversation.

The maintainer communicates in Arabic; the codebase does not. Tel-Agent is a
public AGPL-3.0 project aimed at an international contributor base, and an
Arabic codebase would close the door on outside contributions.

User-facing *interface strings* are a separate matter: those live in
`locales/` and are translated to `en` / `de` / `ar` (see §A4 of the spec).

---

## Rule 1 — Web chat is the first channel; the phone is the last

**Revised 2026-08-22 by D-017.** The original Rule 1 required an answered phone call
before anything else was built. That order is reversed. The superseded text is kept in
`internal/DECISIONS.md`, along with the cost of reversing it.

Milestone 0 is now:

> A visitor types in a web chat, an LLM replies token by token, the reply can be
> interrupted mid-sentence, and the conversation is stored and searchable.

Then the messaging and social channels. **The phone is built last.**

The phone remains the hard case — sub-second latency, barge-in, and no interface to
show what was understood — so the conversation layer is written to its constraints from
the first line even though web chat does not need them. See Rule 3; none of it is
optional. Skipping it means rewriting the layer when the phone arrives, which is exactly
what the original rule existed to prevent.

**What this does not license.** The scope table in Rule 5 still holds, and everything
outside it still goes to `IDEAS.md`.

---

## Rule 2 — Verify each step before moving to the next

Build order inside Milestone 0. Do not start step N+1 before step N works:

| # | Check | How you know it works |
|---|---|---|
| 1 | The number reaches us | The provider console shows the inbound call arriving at our SIP endpoint |
| 2 | Answer the call | You call the number, it stops ringing, silence on the line |
| 3 | Speak fixed text | It answers and says a hardcoded greeting |
| 4 | Hear the caller | Your words appear as text in the terminal |
| 5 | Full loop | You speak → LLM replies → you hear the reply |
| 6 | Take a message | It asks for name and reason, prints a structured result |

Steps 1–2 are plumbing. Step 5 is the product.

**Before any code at all:** buy a number, point it at the SIP endpoint, call it from
a mobile, and confirm in the provider console that the call arrives. If it does not,
the problem is in the number configuration and no amount of Python will fix it.

---

## Rule 3 — Everything streams

Target: **under 800 ms** from end of caller speech to first audio out.

| Stage | Budget |
|---|---|
| Endpointing — deciding the caller has finished | ~200 ms |
| STT final | ~100 ms |
| LLM first token | ~250 ms |
| TTS first chunk | ~100 ms |
| Network / buffer | ~150 ms |

**Endpointing is in the budget and is usually the largest single stage.** A plain
silence threshold of 500–800 ms consumes the entire budget on its own, so semantic
turn detection is required, not optional. A budget that omits this stage is not a
budget — measure it separately from the first call.

The first sentence starts speaking while the rest is still being generated.
**Never wait for a complete LLM response before starting TTS.** This one
decision is the difference between a natural call and an obviously robotic one.

`cancel()` on the TTS provider is not optional. When the caller interrupts,
audio stops immediately and queued speech is discarded. Without it the agent
talks over people and the product feels broken.

Tool latency must be covered by speech: if a tool takes 3 seconds, the agent
says "one moment, let me check the calendar" and runs the call in parallel.
Silence reads as a dropped call.

**If latency is above ~1.5 s, do not add features.** Fix the streaming first.

---

## Rule 4 — Measure from the first call

Log these on every call, starting with the very first one:

- Time from end of speech to first audio out
- Where the time goes: STT final / LLM first token / TTS first chunk
- Interruption handling — what happens when the caller talks over the agent
- German accuracy — at least 20 real calls in Austrian German before trusting
  any STT provider, including names and addresses, which is where it fails

---

## Rule 5 — Scope discipline

Every good idea that arrives mid-build goes into `IDEAS.md`, not into the code.
That file is the mechanism that gets this project finished.

| Tel-Agent owns | Tel-Agent does NOT own |
|---|---|
| Telephony / SIP | General workflow automation |
| Voice pipeline (STT → LLM → TTS) | Integrations with 400 SaaS apps |
| Turn-taking, barge-in | Being a CRM |
| Conversation + memory | Being a PBX replacement |
| Call routing rules | Analog hardware support |
| Transcript archive + search | |
| Tool execution | |
| **Messaging channels** — web chat, SMS, email, WhatsApp, Telegram, Messenger, Instagram, Discord | |

Anything outside the left column is reached through webhooks and the generic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dpro-at/Tel-Agent](https://github.com/Dpro-at/Tel-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
