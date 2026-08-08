---
trigger: always_on
description: DeskMadeline is an exact behavioral and audiovisual port, not a Celeste-inspired
---

# DeskMadeline contribution rules

## Exact-port requirement

DeskMadeline is an exact behavioral and audiovisual port, not a Celeste-inspired
reimplementation. Every change must be cross-checked against the original Celeste
source and assets in `celeste_reference/` and `celeste_graphics_dump/` (neither is in
the repository; `SETUP.md` says how to make them), or against the named mod's upstream
source when implementing mod functionality.

- Port the original constants, state transitions, update order, collision checks,
  input buffers, timers, animation frame timing, particles, colors, compositing,
  sound event paths, sound parameters, and event timing.
- Preserve frame-order details, including freeze frames, component/state-machine
  ordering, coroutine delays, and when effects are created relative to rendering.
- Use the original sprites and original FMOD events. Never add synthesized,
  approximate, placeholder, or "similar" assets or behavior.
- Do not guess when source material can answer the question. Inspect the source and
  document any desktop-specific adaptation in code beside the adaptation.
- Keep desktop-only policy choices (focus gating, monitor boundaries, menus, and
  persistence) separate from the ported gameplay logic.
- If an exact port is blocked by a missing dependency or asset, leave that portion
  disabled and report the limitation instead of silently substituting behavior.
- Verify changes with frame-level or state-level regression checks where practical,
  build Release with zero warnings/errors, and compare observable behavior with the
  original implementation before considering the work complete.

## Behavior is emergent; never special-case it

Most of what a player recognizes as Celeste is not named anywhere in Celeste's source.
Supers, hypers, wavedashes, ultras, cornerboosts, wallbounces and the dream double jump
have no code of their own: they fall out of ordinary systems interacting. Reproduce the
systems and the tech appears by itself.

- When a behavior is missing or feels wrong, find the mismatch in the underlying system
  and fix that. Never add a mechanism whose only purpose is to reproduce an observation.
  A special case reproduces one symptom and diverges everywhere else that system is used.
- Never shadow a vanilla variable with a parallel one. Every site that clears the original
  must also clear the shadow, and a single missed site is a new exploit. Port the vanilla
  field, with the vanilla name, lifetime and clear sites.
- If vanilla stores something as a flag, port it as a flag. Widening it into a timer
  invents a window that vanilla does not have.
- Worked example: the dream double jump is `DreamDashEnd` re-granting the ordinary
  `jumpGraceTimer` on a horizontal exit, even though the exit-frame jump just consumed it.
  `dreamJump` is only a sound selector. An added parallel "dream grace timer" reproduced
  the double jump and simultaneously handed out free mid-air jumps after wall jumps,
  because the wall-jump paths only cleared the vanilla timer.

## The engine is part of the port

Frame counts come from Monocle, not from Celeste's entity code. Porting a `Player` method
without its engine context silently changes input windows.

- `StateMachine.Update` runs the state update *before* the state coroutine, so a state's
  update method runs one extra time on the frame its coroutine finishes.
- `Coroutine.orig_Update` tests the wait timer *before* decrementing it, so a wait ends on
  the frame the timer is no longer positive, not the frame it reaches zero.
- `yield return null` costs a real frame, with the state update running during it.
- Consequence: the dash Super/Hyper window is 12 `DashUpdate` frames. Rewriting
  `DashCoroutine` as a plain countdown loses two of them.
- Update order inside `Player.orig_Update` is load-bearing: the wall boost reads the
  previous frame's `moveX`, retained wall speed is restored before the state machine, and
  the dash floor snap runs after the state machine and before `MoveH`.
- Guard conditions are as much a part of the port as the numbers. `onGround` is only
  evaluated while `Speed.Y >= 0`; `lastClimbMove` is sampled before the slip override.
- Some correct behaviour looks like a bug from the outside. A dream hyper comes out a super
  whenever she reached the block already crouched, because `DashBegin` stands up an airborne
  crouched player and its `else if` cannot crouch her again — which is the whole reason
  Celeste players say not to demo-dash into the block. Forcing the crouch there would "fix"
  the report and break the rule. `DreamHyperChecks` pins both outcomes.
- *When* input is sampled is itself a mechanic. Vanilla refreshes `lastAim` every frame and
  `DashCoroutine` reads it when it resumes, after the freeze - so the aim held about four
  frames *after* the dash press is the one that counts. Latching input at the press frame
  looks like a harmless simplification and silently deletes the hand demo dash: press down
  with dash to duck at `DashBegin`, then swap to a horizontal aim before it is sampled.
  Never snapshot input early "to keep it stable"; sample it where vanilla samples it.

## Constants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solstice23/desk-madeline](https://github.com/solstice23/desk-madeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
