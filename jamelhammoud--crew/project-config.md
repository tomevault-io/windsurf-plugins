---
trigger: always_on
description: Pool LLMs with friends. Open a project and you are in, share it when you want somebody with you, and everyone's local LLM CLIs become shared agents. Chat, docs, and a dashboard, synced through git.
---

# crew

Pool LLMs with friends. Open a project and you are in, share it when you want somebody with you, and everyone's local LLM CLIs become shared agents. Chat, docs, and a dashboard, synced through git.

## Commands

- `yarn dev` — run the app
- `yarn build` — build main, preload, and renderer
- `yarn start` — run the built app, wearing the icon it ships with
- `yarn preview` — the same run, wearing the blueprint
- `yarn test tests/<name>.test.ts` — one suite (vitest), which is how tests are normally run
- `yarn test` — the whole suite, which takes a long time. Only when asked for it
- `yarn tsc --noEmit` — typecheck
- `yarn covers` — draw every music cover on one page
- `yarn selection-check` — select everything in a real window and see what it painted
- `yarn preview-check` — stand a page up in a real window and see what it reached
- `yarn codex-check` — run a real Codex turn and read what came back off it
- `yarn kimi-check` — run a real Kimi turn and read what came back off it
- `yarn dist` — build a mac dmg
- `crew` opens a crew on a folder from a terminal. It ships inside the app, and the This computer page in the settings puts it on PATH. `npm link` in this checkout does the same from here

## Writing

- No em dashes, and no semicolons used in their place. Write plain sentences.
- The app is called Crew, and it is capitalized wherever it is named. It is a name, not a word.
- Plain words. No over-selling, no narration. State things simply.
- UI copy is for everyone, not just engineers. Avoid engineering jargon.
- Copy never echoes the request that produced it. Placeholders and empty states describe what the user can do in general terms, never the specifics of what was asked for or how it was built.
- A heading stands on its own. A line under it saying the same thing again in longer words, or explaining what the field below it is for, is a row of type spent on nothing. Write a second line only when it says something the screen cannot say without it, and never one that starts by naming what the thing is.
- Every line has to say something the screen cannot say on its own. Naming back what a person is already looking at is the obvious half of a bad line, and the flourish tacked on the end is the other: "cursors and all", "and more", "in seconds". Both are filler, and one screen of it teaches people to stop reading the words everywhere else in the app. Take the filler out, and if nothing is left standing, the screen is better with no line on it.
- Never explain the machinery. A platform that will not hand over a key, a model coming down, a fallback that exists because the good path sometimes fails: none of that is the person's to carry, and a line that carries it is the conversation that built the feature read aloud in the product. Write the consequence for the person, or write nothing.
- The skill in `.claude/skills/crew-copy` is the rest of it: the test a line has to pass before it is written, and where each rule came from.
- A line written from the brief reads like the brief. Copy that hands back the parts a feature was asked for, "agents you mention in board chat draw here too", is the request read aloud, and everyone can hear it. Write from what is in front of the person and what they can do next, and name none of the machinery behind it: not the panel, not the mode, not the mechanism, not the model.

## Code

- No code comments, in any file, ever. Humans add comments when they want them. A file already full of them is not an invitation to add another: matching the surrounding style is the one reason that never counts, because every comment in there was written by somebody who wanted it and none of them were written by whoever is reading the diff. Why a thing works the way it does belongs in this file. Why a change was made belongs in the message it was committed under.
- Small files. SOLID. DRY. Readable over clever.
- TypeScript everywhere. React and Tailwind in the renderer.
- Integration tests over unit tests. Every feature ships with coverage of its behavior. Tests live in `tests/` and boot real servers and runners on loopback.

## Design

- Dark mode. Tokens live in `src/renderer/src/styles.css` (Tailwind `@theme`): ink scale for surfaces (`ink-900` background, `ink-800` raised, `ink-700` borders and sunken bars), fg scale for text (`fg`, `fg-secondary`, `fg-muted`, `fg-faint`).
- White is the single action color. `positive` and `danger` appear only for status.
- Type ramp: xs 11, sm 13, base 14, lg 16. System sans. The word "Crew" is set in mono.
- Mono set on the same line as the sans comes out sitting high: the two faces carry their text at different heights, and a row centers boxes rather than baselines. `.mono-inline` in `styles.css` is the pixel that puts them back on one line, and it is what any mono beside sans wears.
- Radii: `rounded-card` (20px) for cards, `rounded-shell` (30px) for the composer. Buttons, tabs, and inputs are pills.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamelHammoud/crew](https://github.com/JamelHammoud/crew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
