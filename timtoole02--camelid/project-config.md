---
trigger: always_on
description: Read this before touching anything in the `fabric` lane. It states what the feature is for, what
---

# Local Backend Fabric — agent handover

Read this before touching anything in the `fabric` lane. It states what the feature is for, what
is already committed, what is deliberately unfinished, and the testing standard the lane is held
to. If you are an agent picking this up, the **Testing standard** and **Traps** sections are the
parts that will actually stop you shipping something wrong.

---

## 1. End goal

**Camelid becomes the honest control plane for every local inference engine on a developer's
machines — including the ones it did not write.**

A developer who already runs Ollama or LM Studio should be able to point Camelid at them and get a
truthful answer to three questions:

1. **What have I actually got?** Which machines, which engines, which models, which of them can be
   served right now — and where the answer is unknown, *that it is unknown*, rather than a zero.
2. **What can each of them actually do?** Tool calls, embeddings, load reporting, warm prefixes —
   and **how we know**: measured here, declared by the API surface, or not checked.
3. **Do they agree with each other?** The same prompt on two engines, with a verdict that is
   allowed to say "nothing can be concluded", and — where the engine exposes it — the chat template
   that explains the difference.

The differentiator is not routing. Anyone can proxy to Ollama. The differentiator is that this tool
**tells you when your backends disagree, and why**, and refuses to state things it has not
established — including about itself.

### Why this matters commercially

Routing to a foreign engine makes us a worse version of that engine: we cannot see its queue, cannot
distinguish its refusals, cannot attest its cache. Telling a developer that their Ollama answers
differently from Camelid on identical weights, and showing them the two templates, is something no
one else does. The capability matrix and the divergence view are the product. Mixed routing is a
convenience built on top.

---

## 2. Parameters of success

This feature is successful when **all** of the following hold. They are deliberately falsifiable.

| # | Success parameter | How it is proven |
|---|---|---|
| S1 | A developer can add an existing Ollama/LM Studio install and see everything it holds, without editing code | live receipt against real engines |
| S2 | Nothing on screen or on the wire is fabricated. An unknown renders as an unknown | ablation: fabricate a zero, a named check must fail |
| S3 | A capability answer is never stronger than its evidence | ablation: credit an unmeasured backend, a named check must fail |
| S4 | The fabric never claims two backends agree, or disagree, without having established it | ablation: attribute a difference from an unstable side, a named check must fail |
| S5 | Default behaviour for existing users is byte-identical | every new feature is opt-in; "unaffected" tests exist alongside every guard |
| S6 | Adding a fourth engine is a new file plus a row, not a redesign | `policy.rs` contains no engine name; verified by grep each phase |
| S7 | Every claim in the docs has a receipt or is marked as not yet established | see "What is NOT claimed" in §4 |

**S6 is checked mechanically. Run this and expect no output:**

```bash
# Production code only. The test module below `#[cfg(test)]` legitimately names
# engines to build fixtures, so scanning the whole file gives a false alarm.
sed -n '1,/^#\[cfg(test)\]/p' src/fabric/policy.rs | grep -nE 'Ollama|LmStudio'
```

If that ever prints, the seam has leaked and placement has started knowing about engines by name.
Verified on this branch: production `policy.rs` is lines 1–787 and names no engine.

---

## 3. Invariants — do not violate these to make something work

These came out of real defects. Each one has caught a bug at least once.

- **I1** Default routing is Camelid-only. Mixed mode is explicit and opt-in.
- **I2** Every answer names the engine that served it.
- **I3** *Unknown is a value.* Never fabricate load, queue depth, capacity or readiness for a
  backend that does not report it. **A zero is a claim.**
- **I4** Affinity is **refused** for a node that cannot attest prefix warmth, never silently
  degraded.
- **I5** A foreign node is **not** tool-capable until measured on that backend *at that version*.
- **I6** No UI element without a backing field in a real response. No status derived from browser
  storage.
- **I7** No control that does not control. If we cannot start a remote process, render the command,
  not a button.
- **I8** Adding a node is always an explicit human action. Discovery proposes; a person disposes.
- **I9** We do not claim answer parity across engines. We measure divergence and display it.
- **I10** No throughput or latency claim without a fresh paired receipt on the exact head.
- **I11** Existing fail-closed transport rules are never loosened to reach a foreign backend.
- **I12** Placement never inspects or rewrites a request body beyond the `model` field.

Two more added by this work:

- **I13** A difference between two nodes is only attributable when **each node has been shown to
  agree with itself**. One run per side proves nothing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timtoole02/Camelid](https://github.com/timtoole02/Camelid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
