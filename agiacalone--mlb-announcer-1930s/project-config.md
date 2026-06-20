---
trigger: always_on
description: Use when the user asks for a 1930s-style radio broadcast transcript, period-accurate old-time announcer call, vintage play-by-play, "Red Barber / Graham McNamee version", or any "make it sound like a 1930s broadcast" transformation of a game. Consumes the Markdown product of the `mlb-game-report` skill (with its structured ATMOSPHERE, SCORING, PLAY-BY-PLAY + Statcast sections) and produces a transcript written in the voice of a Golden Age radio sportscaster, with modern Statcast metrics translat
---


# 1930s Radio Broadcast Transcript

Transforms a structured MLB game report into a transcript of a *live* radio broadcast, as if called by a 1930s-era sportscaster **who has time-traveled to the present and is gamely calling a contemporary game**. Takes the structured product of the `mlb-game-report` skill (PBP + Statcast + atmosphere + captivating-moment data) and re-voices it in period-accurate radio English.

## The conceit

The announcer's voice is always Depression-era. What changes with the source game's date is whether he is **in his own time** or **time-traveling**.

**Always read the `date` field from the source frontmatter before writing and pick the right mode.**

### Mode A — Source date 1925–1949 (his own era)

He is simply calling a game he was scheduled to call. **Drop the time-travel framing entirely.** No "year of our Lord" slips, no marveling at modern players, no "the tracking apparatus tells me" translations — there is no tracking apparatus. There is a press-box telephone, a pair of field microphones, and a guy with a stopwatch. Use the full period vocabulary from `references/vintage-phrases.md` naturally. Period player comps (Ruth, Gehrig, Foxx, Hubbell, Dean, Ott) are natural, not forced. The masthead date matches the source date.

### Mode B — Source date 1950 or later (time-traveler)

He is in our century but keeps his 1930s voice and vocabulary. The humor and charm come from the translation: a Depression-era voice describing a later-era game **fluently but in his own words**. He is informed — he has been here a while, he's been briefed — and he knows what Statcast, Sabermetrics, ABS, replay review, and the pitch clock actually are. But he lacks the native language for them and reaches for period-accurate analogues (tabulating machines, Signal Corps apparatus, photoelectric cells, cinematograph review, the slide-rule school, the log-book boys).

- The announcer occasionally marvels — in character — at modern phenomena: the size and speed of modern players, velocity readings unheard of in his day, night baseball under the lights. A light touch, not a bit every inning. The further from his own era (2020s vs. 1960s), the more he has to reach.
- Modern player names used as-is — Moncada, Trout, Tatis Jr. He can say them confidently.
- **Modern technology and concepts are translated, not avoided.** See `references/translations.md` for the full vocabulary table. Examples:
  - Statcast → "the Signal Corps tracking apparatus" / "the Bell Labs boys with their radio-ranging gear"
  - ABS → "the electric umpire" / "the photoelectric strike zone"
  - Sabermetrics → "the slide-rule school" / "the number men" / "the IBM-tabulator fellows"
  - Replay review → "the cinematograph review" / "the newsreel confab" (never "video review")
  - Exit velocity → "the off-the-bat reading" / "the tracking tells us"
  - Pitch clock → "the electric pitch timer" / "the chronometer"

### Mode C — Source date before 1921 (no radio baseball yet)

Baseball radio broadcasting did not meaningfully exist before August 1921 (KDKA's Pirates-Phillies broadcast). For a game from 1920 or earlier, **tell the user you can't produce a "live radio call" for a game that predates baseball radio**, and offer alternatives: (a) a period-newspaper wire recap via `mlb-game-report`, or (b) a "what it would have sounded like if radio had existed" broadcast with that caveat front-and-center in the transcript. Don't silently produce a live-radio call for a pre-1921 game.

### Always (all modes)

- **LIVE PRESENT TENSE.** This is a radio broadcast happening *right now*, as the game unfolds. The announcer does **not** know what happens next until the ball is hit, the pitch lands, the umpire rules. **Default voice is present tense**: "He swings — *he misses* — strike two!" / "Cronenworth steps in" / "Schanuel is digging in the box" / "Here comes the pitch" / "That ball is jumping off the bat — deep to center — *going, going, gone!*" — not "Moncada homered" or "Schanuel went 3-for-5". Past tense creeps in easily; be vigilant. Between-innings recaps and the post-game sign-off may use past tense for earlier events, but live plate appearances are always live.
- **The source date is the masthead date.** Never substitute a different year.
- **Computers do not yet exist** in his vocabulary (1930s: a "computer" was a person). Avoid the word and its descendants: no "computer", "algorithm", "data", "software", "app", "stream". Prefer "tabulating machine", "records", "the books", "the wire service".
- **Facts from the source, voice from the references.** Every play, count, score, umpire name, pitching change must match the source.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agiacalone/mlb-announcer-1930s](https://github.com/agiacalone/mlb-announcer-1930s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
