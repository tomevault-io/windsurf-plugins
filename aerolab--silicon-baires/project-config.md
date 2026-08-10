---
trigger: always_on
description: Generative 3D with Blender, driven by code, headless.
---

# Silicon Baires

Generative 3D with Blender, driven by code, headless.

## Start here

```bash
./bl scripts/verify_setup.py     # 9 checks, ~3s. If it says 9/9, the environment is ready.
./bl scripts/my_script.py        # run any script inside Blender
./bl                             # open the GUI with the newest .blend
python3 -m ruff check scripts blib   # defects only, not house style. See pyproject.toml
```

The `bl` wrapper finds the Blender binary on its own (or honours `BLENDER_BIN`).

## The rule that is not negotiable

**A render is not validated until it has been looked at.** After rendering, open the
PNG with the Read tool. Framing, exposure and material mistakes raise no exception:
they come out ugly and the script exits 0.

`verify_setup.py` applies the same idea automatically: it measures render luminance to
catch the black or blown-out frame, which is the most common failure mode when working
without a viewport.

## The other rule: count it before you fix it

Three kinds of request come in, and they get three different answers. **Only the
third one is a judgement call**; the first two are decided by what was asked.

**1. A request about one thing. Do that one thing.** "Paint building A red"
means building A. Do not go looking for a rule, do not repaint the city, do not
propose a palette system. This is the case `the .blend is the deliverable`
below is about, and it is the most common one.

**2. A request about everything. Do it once, where the number lives.** "The
buildings are 12 storeys, make them 13" is not eighty edits: it is one, in the
step that decides storeys, followed by re-running the chain. Editing eighty
objects by hand produces a `.blend` that is right and a script that is wrong,
and the next run of that step silently puts all eighty back. **The scope came
from the request — the only thing to get right is not doing it by hand.**

**3. A request about one thing that is a symptom of everything. Do what was
asked, then count, then say the number.** This is the one worth being alert
for, and it is where the rest of this section applies. "There are four people
standing in the middle of the street by the sign" was a true report about four
people, and underneath it were 685 standing on a carriageway and 545 being
driven through during the shot. Four is a nudge. 685 is a rule that does not
exist.

**Do not assume case 3 — measure whether you are in it.** The cost of checking
is one script that counts; the cost of guessing wrong in either direction is
either shipping 681 more of the same, or rebuilding a city nobody asked you to
touch. And when the count comes back large, **the scope is still the user's
call**: report the number and what fixing it costs, rather than quietly widening
the job.

**What gets reported is a sample, not the bug.** Every defect in this project's
history arrived as one instance somebody happened to see, and every one of them
turned out to be a rule that was wrong everywhere it applied:

| what was reported | what it actually was |
|---|---|
| four people standing in one street | 685 of 2883 on a carriageway, 545 driven through |
| a tree in a wall | 917 of them |
| the roof signs flicker in the browser | 100 of 118 closed meshes wound inside out |
| two cars in the same spot | 7 pairs, one at a separation of exactly zero |
| one zebra crossing that led nowhere | all 216, from subtracting `WALK` twice |
| the masts are floating over the roof | all 13, from standing on the parapet line |
| the traffic looks odd on that street | one whole axis of the city driving on the left |

So the first move on any defect is **not** to fix it. It is to write the thing
that counts how many there are — and that number decides what kind of problem
it is. Four people in a street is a placement to nudge. 685 is a missing rule,
and nudging four of them ships the other 681.

The loop, in order:

1. **Measure.** Count every instance before touching anything. The count is the
   scope, and it is usually one to three orders of magnitude larger than the
   report.
2. **Find the rule that is wrong**, not the instances. 685 people on the asphalt
   was one fact nobody had written down: the placement had no idea what a road
   was. It could not be fixed 685 times.
3. **Fix it where the rule lives**, once, and make every caller read it. Look
   for the second copy while you are there — most of these were two places
   disagreeing about one number.
4. **Leave the counter behind as a check**, and make it ask a question the fix
   cannot answer by construction. See `91_check_crowd` on why it counts people
   a vehicle drives through rather than people standing on asphalt: once the
   placement reads the geometry, asking the geometry is circular.
5. **Re-measure.** The number is 0, or the fix is not done.

**The test that tells case 3 from case 1** is not "could this be generalised" —
almost anything can. It is **"is this instance a violation of a rule that should
hold everywhere?"** A building that reads badly from the hero camera breaks no
rule: it is taste, taste does not generalise, and there is nothing to count.
A building standing inside another one breaks an invariant, and there were
never one of those. **Ugly is a case. Wrong is a rule.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aerolab/silicon-baires](https://github.com/Aerolab/silicon-baires) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
