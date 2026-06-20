---
trigger: always_on
description: Use whenever the user wants to create narrative video content — a reel, short film, movie, music video, trailer, promo, teaser, episode, or multi-episode series — from a story premise or idea. Triggers on phrases like "make me a reel", "create a short film", "generate a movie", "I want an episode about", "turn this into a video", "make a cinematic clip", or any request where a story or scene is described and video is the expected output. Always use this skill for narrative video work; it handles
---


# Film Maker

Turn one sentence into a complete piece of cinematic video. Handles reels, short films, episodes, and multi-episode series in any genre — drama, thriller, noir, comedy, sci-fi, soap opera, fable, anything.

You (Claude) do the creative work — writing the plot cinematically, designing the cast and world, breaking the story into shot-level beats. Then you run a deterministic production pipeline that calls `gen-ai` (Picsart's CLI) for images and video, and `ffmpeg` for concatenation. The user gets a finished mp4.

## Why this skill exists

Generic "prompt a video model" workflows skip the hard part: professional writing, cast consistency across shots, cinematic camera language, emotional pacing. This skill bakes those in. Without this, characters change face every beat, plots feel like AI slop, and the video model gets asked to invent everything from a single flat prompt.

## Prerequisites — check before starting

Run this check silently before the first generation call. If anything is missing, tell the user what to install and stop.

```bash
node --version                         # must be v22 or higher
command -v gen-ai                      # must resolve
gen-ai whoami                          # must show an authenticated user
command -v ffmpeg                      # must resolve
command -v jq                          # must resolve (for parsing gen-ai --script output)
```

If `gen-ai whoami` fails, tell the user to run `gen-ai login` or set `PICSART_ACCESS_TOKEN` + `PICSART_USER_ID`.

## The workflow

The skill is an **eight-step pipeline**. Do them in order. Steps 1–6 are creative + planning work you do yourself. Step 7 runs the generation scripts. Step 8 delivers.

**When the user invokes the skill with `--plan-only` or says "just plan, don't generate yet", stop after Step 6.** See the `--plan-only` section below.

---

### Step 1 — Understand intent

Extract from the user's message:
- **Premise** — the story seed
- **Format** — reel (vertical short, 10-60s) / short film / feature / series? If unclear, infer from phrasing ("reel" → 9:16 short; "movie"/"film" → 16:9 longer)
- **Genre + tone** — see `references/genre-catalog.md` for the 14 core genres and tone modifiers. If the user didn't specify, propose 1–2 candidates from the premise and ask
- **Length** — seconds, minutes, episode count (episodes × length-per-episode)
- **Aspect ratio** — infer from format (reel → 9:16, film → 16:9, square post → 1:1), or ask. Video models only support 9:16 / 16:9 / 1:1 — don't offer others
- **Episodes vs continuous** — if it's more than ~3 minutes of content, ask: continuous single film, or split into episodes? Episodes unlock arcs and cliffhangers; continuous is simpler
- **Project name** — short slug for the output folder (e.g. `noir-alley`, `mars-gardener`)

If any of these are unclear after reading the premise, ask exactly the needed clarifiers (not a bulk form — pick the 1–3 that matter most for their request).

Create the project folder at `./<project-name>/` relative to current working directory unless the user specifies a different location. Write `project.json` from `templates/project.json.template` with the resolved settings.

---

### Step 2 — Write the plot

This is where amateur AI video generation dies. Spend real thought here.

**Load the right playbook.** Read `references/genre-playbooks/<genre>.md` for structural rules. For blends (e.g. sci-fi thriller), read both and merge: primary genre drives act structure + pacing, secondary genre contributes tonal texture + setting logic.

**Write the plot as `plot.md` in the project folder.** Structure:

```markdown
# <Title>

## Logline
One sentence. Who wants what, what stands in their way, what's at stake.

## Act I — Setup
## Act II — Escalation
## Act III — Resolution

## Character arcs
Per character: where they start emotionally, where they end, the turning point.

## Tone and visual texture
Color palette, lighting philosophy, shot rhythm.
```

**Craft rules — apply them consciously:**

- **Start with a hook in the first 3 seconds.** Short content = no ramp. Even features open on a question or an image that can't be ignored.
- **Emotional contrast.** Tension → release → tension. No flat plateaus. The rule of 3 (setup, reinforce, subvert) works at every scale.
- **Subtext over exposition.** Characters don't announce their feelings — they show them through action, what they don't say, what they misread. If a character tells another character what they already know, that line needs to die.
- **Motivation drives action.** Every beat happens because someone wants something. "And then X happens" without a why is AI slop rhythm.
- **One memorable image per act.** A recurring visual that carries meaning — the vial, the photograph, the dying seedling, the neon reflection. Viewers remember images, not dialog.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hunanyanr/film-maker](https://github.com/hunanyanr/film-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
