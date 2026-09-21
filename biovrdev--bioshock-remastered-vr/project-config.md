---
trigger: always_on
description: Native VR mod for BioShock Remastered (`BioshockHD.exe`, 32-bit, D3D11, UE2.5
---

# BioShock Remastered VR — working notes for Claude

Native VR mod for BioShock Remastered (`BioshockHD.exe`, 32-bit, D3D11, UE2.5
fork with no reflection system). Everything here was found by scanning and
measuring, not read out of an SDK.

## Read this before touching code — the context ladder

> **Uncertainty is a signal to load more context, not to guess.** Reading a doc
> costs tokens. Guessing costs a headset cycle, and this project's entire
> graveyard is made of confident guesses. When in doubt, escalate and say you did.

**Rung 1 — always.** This file (it auto-loads, so it is free) plus your session
card: `.planning/sessions/M<n>.md`, the section for your session. The card is your
task. `.planning/ROADMAP.md` says which one is next.

> **`.planning/` is git-ignored, and if you cloned this repo you will not have
> it.** That is deliberate: the planning documents describe features well before
> they exist, and this project announces a feature when it works rather than when
> it is designed. Nothing is withheld from the record — every measurement
> graduates into `docs/` in the commit that uses it, which is where you would look
> for it anyway. **`docs/` is the published knowledge base and is complete on its
> own.** Working from a clone, start at `docs/CODEMAP.md`.

**Rung 2 — the default for any code session.** Plus the `docs/modules/*.md` the
card names, the `docs/INVARIANTS.md` section it names, and `.planning/STATE.md`
§ *Next step*. A session that reads Rung 2 and makes an anchored edit is behaving
correctly and needs no permission to go further.

**Rung 3 — escalate freely.** No asking, no apology; just note which rung you went
to and why.

| When | Load |
|---|---|
| Touching an offset, address or pattern scan | `docs/ENGINE-MAP.md` + `engine-offset` skill |
| An anchor isn't where the doc says, or a doc contradicts the source | The module doc, then the source region — **and report the drift** |
| An idea occurs that the card didn't specify | Full `docs/INVARIANTS.md`, both lists, before proposing it |
| Cross-module work, or you can't tell which module owns something | `docs/CODEMAP.md` |
| Writing more than a few lines of new code | `docs/STYLE.md` — no auto-formatter, deliberately; never bulk-reformat |
| A setting seems inert | `ini-check` skill + `docs/modules/config.md` |
| Reading a log or diagnosing a play session | `log-triage` skill |
| "What does the game actually do here?" | `docs/UNREALSCRIPT.md`, then grep `research/uscript/` |
| Two rungs in and still unsure about behaviour | Read the source region |

**Rung 4 — subagents. Only when the arc has genuinely dead-ended** and a fan-out
search across many files would actually help. A cold subagent re-derives context
you already hold; it is the most expensive move available here.

Avoid whole-file reads over ~400 lines — `CameraHook.cpp` (3759) and
`DrawHook.cpp` (2161) are ~60k tokens together. Grep the banner anchor and read a
window. If the window genuinely isn't enough, read more; under-reading is worse.

**Current arc:** live state → cutscene detection → QOL. Findings and design in
**`docs/ARCHITECTURE.md`**. The long-held premise that this fork has no usable
reflection is **false** — read it before proposing anything about reading engine
state.

## The graveyard — check this before proposing anything

One line each; evidence in `docs/INVARIANTS.md` § *Falsified*. **Every one of these
is plausible enough to be proposed again — that is why it is here.** It lives in
this file so the check costs nothing.

*Cutscene detection:*
1. **`myHUD.bHideHUD`** — the offsets are right (`controller+0x71C`, bool DWORD `+0x490` bit 0, back-reference confirmed). The DWORD **never changed once** in 16 minutes across bathysphere, plasmid, rescue and combat — and did not move even while the HUD visibly appeared and disappeared. Suspect `HideMovie('HUD')` instead.
2. **ViewActor divergence** — never leaves the pawn; `+0x450`/`+0x620`/`+0x914` track it for whole sessions.
3. **Pitch-rate latch** — latched during ordinary combat for four straight seconds.
4. **Pitch servo** — runaway feedback loop; froze the view and the hand.
5. **S75/S78/S79 render-side unwind** — made scripted sequences worse.
6. **Cached view-target scans** — no signal.
7. **`LastPlayerInputContext` on the pawn** — window correct, has *never* locked. (The **controller** copy is untried — `docs/ARCHITECTURE.md` finding 3.)
8. **Console `get`** — returns the class default object, not live state.
9. **Input-ignored detector** — sound, but needs the player to push the stick, so it is silent when a cutscene starts standing still.

*Aim and movement:*
10. **`AimSource=2`** cannot exist — the game's heading freezes permanently.
11. **Body-follow yaw servo** — ported from the reference mod; did not feel right.
12. **`ModYaw` alone** — zeroing `sThumbRX` freezes `Controller.Rotation`, which forced-move sequences steer by. The opening bathysphere walks into the back wall.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BioVRDev/Bioshock-Remastered-VR](https://github.com/BioVRDev/Bioshock-Remastered-VR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
