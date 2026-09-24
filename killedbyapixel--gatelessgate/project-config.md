---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An interactive edition of the Mumonkan (The Gateless Gate) in the browser: all 49 koan cases staged as small low-poly ink-painting dioramas, with text, narration, ambience, and a meditation timer. It is an **interactive book, not a game** — dioramas are ambient scenes; touch responses are optional delights, never gates.

The design doc at `docs/gateless-gate-design-doc.md` is authoritative; its dated revision notes **override** anything they contradict elsewhere in the doc. Milestone plans/specs live in `docs/superpowers/`. **`docs/` is gitignored**, like `local/` — it is the working record, not part of the published repo, so a fresh clone will not have it and nothing that ships may depend on it.

## Commands

- Run: `npm start` (`npx -y http-server -p 8105 -c-1 .`) then visit http://localhost:8105 — **no build step**; ES modules are served directly and Three.js is vendored in `lib/` (version in `lib/THREE_VERSION.txt`).
- Test all: `npm test` (runs `node --test` over `tests/`, Node 20+)
- Test one file: `node --test tests/k29.test.js`
- Regenerate the whole-book Markdown at the repo root: `npm run book` (`node scripts/build-book.js`) — writes `THE-GATELESS-GATE.md` from the generated text modules, so run `build-text.js` first if the source changed. `tests/book-md.test.js` fails if the committed file is stale.
- Regenerate the text modules after editing `book/gateless-gate.md`: `node scripts/build-text.js` (writes both `src/koans/text/mumonkan.js` and `matter.js`). **The whole book is that one checked-in Markdown file** — the preface, the forty-nine, the afterword: `##` opens a page, `###` opens a section, and section headings are matched **exactly**, because a heading chooses the key that names the baked narration file and a retitled heading would otherwise orphan an mp3. Inside a section a blank line is a paragraph or stanza break; within a paragraph, prose line breaks are wrapping and rejoin with a space, verse line breaks are the verse and are kept. `scripts/lib/parse-book.js` holds the manifest saying which sections are which — that table replaced the U+3000 ideographic space the old matter source marked verse with. `book/translation-notes.md` beside it is the scholarly record behind the new front-and-back-matter translation (Chinese, Taishō citations, unresolved readings); nothing builds from it, and the four pieces that ship point at the book file rather than repeating it.
- Re-bake narration after a text change: `node scripts/build-narration.js` — only units whose text/voice/preset hash changed are regenerated, so this is normally a few files and a few cents. Needs a Gemini key in `local/gemini-key.txt` or `GEMINI_API_KEY` (the live provider is Gemini — see `PROVIDER` in `scripts/lib/narration-voice.js`; the OpenAI path remains for the audition bake-off). `--dry-run` shows what would bake without spending; `--case N` — or a matter-page slug such as `preface` — scopes it, and an unrecognised value is a hard error, never a silent fall-through to baking the whole book. Voice and delivery live in `scripts/lib/narration-voice.js`; `scripts/narration-audition.js` is the bake-off for changing them.
- Audit a bake for bad takes (no API cost): `node scripts/check-narration.js` (duration vs character count) and `node scripts/check-narration-wps.js` (speaking pace, crediting pauses). They normalise differently, so run both. Neither catches a same-length garble — that still needs ears.
- Screenshots while the preview panel is hidden: `node scripts/dev/shot-server.js` (port 8106; workspace launch config `gate-shots`), then POST a `canvas.toDataURL(...)` string to `http://localhost:8106/<name>` — files land in `shots/` (gitignored).

`local/` is gitignored — it holds the source text and the author's local notes; don't assume its contents exist in a fresh clone.

## Headless driving

`window.gate` exposes hooks for driving the app without a visible page: `step(n)`, `state()`, `enter(slug)`, `exit()`, `menu(open?)`, `skipIntro()`, `dissolve('in'|'out', s?)`, `sit(min)`, `endSit()`, `markRead(slug)`, `markSat(slug)`, `setSound(on)`, `look(on?)`, `readAloud()`, `auto(on?)`. `state().reading` reports `{ look, all, slug }` — `look` is the text-aside view, `all` whether the page is being read aloud. The look never starts a reading, and a reading stops at the end of its own page rather than turning over.

Caveat: the hidden preview panel pauses `requestAnimationFrame`, so `await gate.enter(...)` hangs — dissolves only advance inside `step()`. Fire the call, then `step(60)` once per await stage (`loadKoan` → dissolveOut → dissolveIn), yielding between batches so microtasks flush.

## Architecture

- **Determinism rule:** seeded noise everywhere (`src/util/noise.js`), **no `Math.random` outside `src/audio/**`** — same steps, same state. Audio is exempt because noise buffers don't affect sim state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KilledByAPixel/GatelessGate](https://github.com/KilledByAPixel/GatelessGate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
