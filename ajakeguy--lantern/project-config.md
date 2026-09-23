---
trigger: always_on
description: > **Standing instruction to Claude Code: this file is living documentation.**
---

# LANTERN — Project Constitution

> **Standing instruction to Claude Code: this file is living documentation.**
> You are responsible for keeping it current. At the end of any session that changes
> architecture, adds/modifies a game, changes the phonics scope, or makes a design
> decision, update the relevant section AND the Build Log below — without being asked.
> If a decision in-session contradicts this file, flag the conflict, resolve it with
> the developer, then record the outcome here. This file is the single source of
> truth; no context should need to be passed in from outside it.

*(Name settled: **Lantern**. GitHub repo: `Lantern`.)*

---

## What this is

A web app (React, hosted on Vercel) that builds advanced early literacy skills for
kids roughly ages 3–8. Built by a parent, for their kids first, with care and craft.
Not an edu-tech product clone.

## Philosophy — the part that must never erode

**Reading reveals the world.** Words are not a school subject bolted onto life;
they are how a person illuminates what was already there. A lantern doesn't create
the room — it shows it to you.

Operationalized as one design law:

> **Reading must have consequences. Never "read this, get a point" —
> always "read this, and something becomes possible that wasn't."**

Decode a word → the thing appears. Read a note silently → you know a secret nobody
said out loud. Read a sentence → you can build the scene it describes. Every
mechanic must pass this test before it ships.

### Two strands, one roof

- **SOUND** (learning to light the lantern): phonemic awareness → letter-sounds →
  blending → automaticity. Audio-forward, out-loud, phonics-driven.
- **LIGHT** (learning to see by it): silent reading, comprehension, meaning,
  stamina. Deliberately audio-restrained — silent reading is the point.

The app shifts weight from Sound to Light as the child progresses. Branding and
parent-facing copy should always convey both halves ("learning to light it /
learning to see by it"). The name must never read as phonics-only.

### Hard rules (violations are bugs)

1. **No mascot, no guide character, no digital figurehead.** The world teaches
   through its mechanics. The child is the protagonist. The only correspondent in
   the app is a real human (see Family Post). Do not add a helper character, a
   talking animal, or a narrator persona, even as "just a small touch."
2. **No engagement-farming.** No points, streaks, confetti storms, daily-login
   rewards, countdown timers framed as pressure, or variable-reward loops. The
   reward system is the Field Journal (below) and the feeling of capability.
3. **Errors are never punished.** A wrong blend makes the wrong (funny) thing, not
   a buzzer. Wrong comprehension leads somewhere silly, not to a red X.
4. **Screen is a springboard, not a destination.** Off-screen writing and offline
   games are first-class features, not extras. The ✎ handwritten-word mechanic and
   the Grown-ups offline games must survive every redesign.
5. **Nonsense words are sacred.** Decodable non-words (zib, mip) summon collectible
   creatures. This is disguised assessment — a child reading nonsense is provably
   decoding, not memorizing. Never remove or trivialize it.
6. **No third-party analytics, ads, or accounts beyond what hosting requires.**
   This is for children.

## The reward system: the Field Journal

Every word a child reads becomes a specimen card. Cards grow in with re-reading:
**sketch → inked → full color** (4 clean reads). Counters track *sounds met*,
*specimens*, *fully inked* — capability measures, never scores. Cards carry a ✎
toggle: "I wrote this by hand." Shelves: Things / Odd Menagerie (nonsense
creatures) / Words / Wild Words. The journal is the child's owned, growing proof
of their own knowledge. Procedural creatures are deterministic per word (same word
= same creature, forever).

**The Grove** is the journal's inhabited twin: every summoned thing and creature
lives permanently in one home scene (first tab; the boot-default tab once any
specimen exists). Creatures render at their journal stage — re-reading a word
visibly grows its creature in the world. Populated entirely from `specimens{}`;
no separate state. Tap a resident → it says its name.

## The ladder — games by band

Bands unlock by demonstrated skill, never by age. Overlap is expected.

### Band 0 — Ears only (~3–4) · Strand: Sound
No child-facing screen games. The app coaches the PARENT with offline games tuned
to progression: Sound Chain, robot-talk blending ("c...a...t — what did I say?"),
rhyme hunts, Menu Hunt, Silly Sentences. Phonemic awareness with zero letters is
the strongest predictor of reading success; it belongs off-screen.

### Band 1 — Cracking the code (~4–5) · Sound-heavy
- **Letter Safari** — the lantern mechanic: scenes are dark; a draggable light
  reveals letters camouflaged into naturalist SVG artwork; tap a lit letter →
  sound, and it morphs into its mnemonic. Once all letters are found, the
  **echo round** reverses the question (hear a sound, find its letter) and
  masters the scene (`scenesMastered`). Darkness is quiet, never a buzzer;
  keyboard focus counts as light. [BUILT v1]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajakeguy/Lantern](https://github.com/ajakeguy/Lantern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
