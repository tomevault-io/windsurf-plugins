---
trigger: always_on
description: User reads replies on a phone. Keep answers **simple, short, precise**: lead with the result, no preamble/recap, thought process in 1-3 terse bullets, prefer bullets over paragraphs. Same for subagent prompts (task + constraints tight, no narrative). Long detail only on explicit request.
---

# Project: FruitNinja.exe Reverse Engineering & Port

## Response style — short
User reads replies on a phone. Keep answers **simple, short, precise**: lead with the result, no preamble/recap, thought process in 1-3 terse bullets, prefer bullets over paragraphs. Same for subagent prompts (task + constraints tight, no narrative). Long detail only on explicit request.

## Port Goal
- **Fidelity first** — match the original game as closely as possible
- Preserve all gameplay mechanics, physics, scoring, timing, and visual behavior
- Preserve simultaneous multi-finger slicing (per-finger blades, up to 8 fingers) — this *is* the binary's "multiplayer". NOTE: v1.6.1 has **no** same-screen split-screen mode to port; its networked MP was **online P2P (defunct, stubbed)**, and `m_PlayerIdx`∈{0,1,2} is the P2P/EntityTracker partition, NOT a screen-half split (confirmed by the input-path audit, #158).
- All UI screens and widgets should be ported, not just core gameplay

## Defunct features — stub, never skip

Defunct features (OpenFeint, GameCenter, P2P multiplayer, online news, online leaderboards, etc.) are **stubbed**, never skipped. The shape — class, struct fields, vtable, public-API methods, call sites — is **preserved as if the feature were ported**, with method bodies as no-ops that return safe defaults. Each stub carries `// Defunct: <subsystem> — no-op stub; v1.6.1 <Symbol> @ 0x<addr>` so it's greppable.

Why: skipping breaks the call graph and causes cascading port-side workarounds at every call site. Stubbing keeps the call graph identical to the binary, isolates the "this is a no-op" decision to one spot, and lets the asm-verify pass treat the call sites as cosmetic-only divergences rather than real bugs.

What "stub" means concretely:
- Class exists, vtable layout matches binary (slot order + count).
- Public methods are declared and have bodies. Bodies typically return `0` / `false` / `nullptr` and do nothing observable.
- Singletons' `GetInstance()` returns a valid (possibly empty) instance so call-graph-walking code doesn't crash.
- Network packet structs / handler interfaces are declared so call sites compile.
- A `// Defunct: ...` comment marks each method body.

What "skip" would have meant (NOT what we do): omitting the class entirely so call sites in surrounding code need to be deleted or guarded with `#ifdef`s. Don't do this.

**Defunct UI is still DRAWN — when the target binary actually draws it.** Stubbing the *logic* never means dropping the *visuals*. If a defunct feature has in-game UI that **the v1.6.1 target's call graph actually instantiates and draws** — a button, a navigable page in a carousel, a badge, a banner, a menu entry — port that UI as a **visible stub**: it must appear on screen exactly as the original drew it (same texture, position, layer), even though tapping/activating it is a no-op. The handler is a no-op; the draw is faithful.
  - **The test is "does v1.6.1 draw it," not "does the class exist."** A defunct class that is *linked-but-unreferenced* in v1.6.1 (zero call-site xrefs — present only as RTTI/vtable relics from other SKUs/older builds) is **dead code, not in-game UI**. Do NOT instantiate it to force visuals the v1.6.1 binary never shows — that's a band-aid that *adds* divergence. (Concrete case: the game-over fact-board nav arrows. They only appear when >1 page is registered; v1.6.1 registers exactly one page per mode and the multi-page classes — `FruitFactLeaderboard`/`FruitFactRewardsPage`/etc. — have no call sites, so v1.6.1 shows **no arrows**. The port correctly shows none. Restoring them would be a `// DIFFERS:` enhancement, not fidelity.)

## No band-aid fixes

When a port-side bug is identified, the fix is to **RE the binary correctly and port it faithfully**, NOT to apply an empirical workaround that happens to make the visible symptom go away.

Forbidden patterns:
- **Magic-number fudges** — `pos.y - 20` because "it looked right", `scale * 0.7f` to compensate for an oversize bug, hardcoded offsets that aren't in the binary.
- **Symptom-masking branches** — `if (problem_value) { force_correct_value(); }` instead of finding why the value is wrong.
- **Empirical try-this-might-work patches** — guessing at fix values without RE'ing the binary's actual semantic.
- **"Good enough for now" workarounds** that drift from binary behaviour without an explicit `// DIFFERS:` marker explaining why fidelity is intentionally sacrificed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mariotaku/FruitNinja](https://github.com/mariotaku/FruitNinja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
