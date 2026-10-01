---
trigger: always_on
description: A native Linux music player: C++20, Qt 6 Quick, CMake and Ninja. Material 3
---

# Sung

A native Linux music player: C++20, Qt 6 Quick, CMake and Ninja. Material 3
throughout. Targets CachyOS on Wayland.

These are hard rules, not preferences. If one of them makes a task larger than
expected, the task gets larger. Say so and do the work; do not quietly drop a
rule to finish sooner.

## Design

Google's Material 3 documentation is the source of truth for every visual and
interaction decision. Not a reference, not an inspiration. When a component,
measurement, colour role, type role, shape, state or motion pattern is
specified there, the implementation matches it, and the comment in the code
says which rule it is following and why.

m3.material.io is a JavaScript application: a plain fetch returns an empty
page, though a real browser renders it, and its guidelines pages carry the
rules. The numbers live in the generated token files under
`androidx/androidx/compose/material3/material3/src/commonMain/kotlin/androidx/compose/material3/tokens`,
and the component's Compose source beside them. Read both. The token files
declare values as `inline val X get() = Y`, so a search for `val X =` misses
them. The tokens are
generated and are sometimes knowingly wrong, and the source says so in a
comment when they are. A value quoted from memory is not research.

The Material alignment passes over the existing interface are finished. Do
not start another one unprompted; new work follows the specification as it
is built.

Where the specification and an existing implementation disagree, the
specification wins and the change is made. Where Material offers a choice
between valid options, pick the one the guidance recommends and state the
reason.

The interface is minimal and stays minimal. No decorative text, no headings
that restate what is already obvious, no sections that exist to fill space, no
controls added for completeness. Material's density and spacing are the floor,
not an invitation to spread out. Minimal and Material are not in tension here:
Material's own guidance is to earn every element.

Minimal is not timid either. When a control is to become more Material, start
from Material's own example for the same use, and offer an Expressive option
(shape, motion, button group behaviour) beside the conservative one. A change
that only resizes what is already there misses the point.

The aesthetic that exists now, including the animations, is deliberate and
stays. Refactoring, tidying or de-risking work never arrives at the cost of
how the application looks or moves. If a structural improvement would change
the visuals, raise it first.

## A feature is not built until all of this is true

1. **Real automated tests.** Behaviour is verified against the real thing, not
   a mock. A test that would still pass with the feature removed is not a
   test.
2. **A simulated user test.** The feature is driven through the interface the
   way a person drives it: real clicks at real coordinates, real key presses,
   real waits for real state. Calling the backing function directly does not
   count as exercising the feature.
3. **A visual capture.** The stage takes screenshots of the feature in its
   states, and those screenshots are looked at, not merely produced.
4. **A UI and UX audit.** Check what the change does to everything around it:
   layout at narrow and wide windows, focus order, keyboard reach,
   accessibility roles and names, disabled and empty states, light and dark,
   reduced motion. A feature that works and degrades its neighbours is not
   finished.
5. **Measurements.** CPU while idle, resident and proportional memory, binary
   size, and fluidity under interaction. Compare against the figures before
   the change. A regression is a defect unless it is stated, justified and
   accepted.
6. **A run on the real desktop.** The harness is offscreen and software
   rendered, so it never exercises the scene graph the person actually sees.
   Launch the built binary on the compositor at least once, with an isolated
   profile so the owner's library is untouched, and photograph it with
   `grim`. Launch it where it does not take over the screen of someone using
   the machine. A change is not confirmed until it has been seen outside the
   harness. Where there is no compositor, as in a cloud session, say that
   this step is still owed.
7. **Documentation.** Record the Material rules the feature rests on and the
   framework guidance it follows, in the code next to the thing they govern.
   A number copied from a specification is meaningless without the sentence
   that says where it came from.

Report what actually happened. If a stage failed, quote it. If a step was
skipped, say which and why. Never describe work as verified when it was not
run.

## Motion

Material describes motion as springs, published as a damping ratio and a
stiffness. `src/m3motion.cpp` solves each one and hands Qt the curve and the
duration it implies; `Theme.qml` exposes them as the six spring tokens. Every
animation reads one of those. A hand-written duration, an easing curve chosen
because it looked right, or a Qt `SpringAnimation` with invented constants is
a defect, and the numbers drift a long way when they are: the curves these
replaced overshot by a third where the physics asks for a sixtieth.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yappologistic/Sung](https://github.com/yappologistic/Sung) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
