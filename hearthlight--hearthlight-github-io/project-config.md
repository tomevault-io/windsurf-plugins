---
trigger: always_on
description: A cozy 2.5D pixel-art life game (solo story mode whose valley opens onto the wild lands + a
---

# Hearthlight — working notes for Claude

A cozy 2.5D pixel-art life game (solo story mode whose valley opens onto the wild lands + a
local **Party Mode** for 1–8 players with phones as controllers; a phone can drive the solo game
too). Plain ES modules, **no build step**, Three.js 0.170 from the
jsDelivr CDN (import map in `index.html`). Everything is procedural: every texture, model and
sound is made in code — never add image or audio files.

The user writes in French or English, expects a very high bar (score each area /10, iterate
to 9+), judges from in-game screenshots, and wants finished work committed & pushed to
`origin/main` (message ending with the `Co-Authored-By` line).

## Run & test

```bash
python3 tools/devserver.py 8765        # static files + /__shot + /__lan + /ws party relay
```

- Game: <http://localhost:8765/?debug=1> · phone controller: `/pad.html#CODE`.
- `window.game.debug`: `step(n, dt)`, `shot(name, scale, crop)` (PNG in `screenshots/`,
  git-ignored), `newGame`, `tp(x, z)`, `hour(h)`, `pause(on)`.
- `const T = await import('/tools/partybots.js')`: `boot(n)` (title → party lobby with n bot
  phones), `mode('explore'|'waves'|'brawl'|'story')`, `vote(re)`, `fight(frames)`,
  `warp(x, z)`, `clearWave()`, `skipTalk()`, `autoplay('lobby')`. Bots talk to the real relay.
- `window.__errs` (filled by `T.boot`) and the console must stay at **0 errors**.
- Translations: `node tools/i18n-scan.mjs` must report **0 missing** in every language (fr, es,
  de, it — and each one's party lines; `--list` prints them, `--lang=de` one language);
  `node tools/i18n-check.mjs es` compares a language's files with their French twins. Screens in a
  language: `tools/langshots.js` (`start('de')`). Syntax check: `node --check file.js`.
- Remote play (`play.html#CODE.key`, a friend at home) needs the Node relay — the Python dev
  server has none: `node server/relay.mjs --static . --lan --host 0.0.0.0 --port 8792` (once
  `npm --prefix server ci`). Each remote player gets their own camera (`p.rcam`, drawn by
  `Party.drawRemote` into `remote-host.js`'s canvases); the big screen frames the others.
- Test Party Mode with 1, 4 and 8 bots; check the solo game still works and the whole story
  in `T.autoplay()` (the first bot wears the crown and starts the party: `{t:'start'}`).
- Solo quick start: `D.pause(true); await D.newGame('Alex'); await D.skip(200);
  game.state.flags.wildIntro = true; game.world.wild.chooseClass('mage'); D.tp(-43, 86)`
  (a steppe camp; `game.world.wild` is the `Wild`, `.combat`, `.encounters`, `.travel`…).
- A phone in solo: `game.phone.start()`, then open `/pad.html#CODE` in another tab; the pad
  records drawing errors in `pad.S.drawError` (its loop never stops on one).
- Gamepads (the pane has none): `const F = await import('/tools/fakepad.js'); F.install(2,
  ['xbox', 'ps'])` fakes `navigator.getGamepads()` (families xbox / ps / nintendo);
  `F.press(i, 'a')`, `F.set(i, 'select', true)`, `F.stick(i, x, y)`; rumbles land in
  `window.__rumble`. `tools/padparty.js` `start(bots, shots)` plays Party Mode with pads (join,
  their big-screen menu, a queue, a late pad, rumble): poll `window.__pp`.
- Trailer (English, for Reddit): `tools/trailer.js` scripts the Party Mode shots, `tools/reel.js`
  records them frame by frame at 1080p (`POST /__rec` → ffmpeg, lossless mkv in
  `screenshots/reel/`) with captions in the pixel font, a real `pad.html` phone and the game's own
  sounds re-rendered offline; `tools/reelcut.py` cuts them after `tools/trailer.edl.json`. The
  recipe is in `trailer.js`'s header (`TR.all()` ≈ 10 min, `TR.mix()`, then `reelcut.py cut`).

### Known pitfalls

- **Never name a variable or parameter `t`** in a function that calls `t()` (the i18n
  function). It has broken things before.
- In tests, wait until `game.world.overCol` exists before `T.boot()`.
- Don't `await` a party fade (`fadeTo`, `backToLobby`) without stepping `game.debug.step`.
- Bots need a real `setTimeout` delay between steps so the relay delivers their messages.
- When the browser pane is hidden, `requestAnimationFrame` is paused: drive frames with
  `game.debug.step`, and capture the phone with `window.pad.draw()`.
- The big world's ground is painted in workers: inside a tight `game.debug.step` loop their
  messages never arrive — yield real time (`await new Promise(r => setTimeout(r, 100))`)
  before screenshots of new places.
- Navigating a tab to the same `pad.html#CODE` URL doesn't reload it (hash change only): use
  `location.reload()` to pick up new code.
- Editing with scripts: replace exact strings and check they occur once — never cut a slice
  between two markers (it silently deleted half of party.js once).
- `javascript_tool` times out after 45 s: split long scripts or run them in the background
  and poll a result variable.
- After a test with a real pad tab, that phone can come back into later `T.boot()` parties
  (even with its tab closed) and wear the crown, so `T.autoplay()` never starts: remove it
  (`P.removePlayer`) or crown a bot (`P.host.give(P.players.find((p) => p.id === 'bot0'))`).
- A Bash hook blocks heredocs containing the bare word "helm".
- The i18n scanner reads a straight apostrophe in a `//` comment after code as the start of a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hearthlight/hearthlight.github.io](https://github.com/Hearthlight/hearthlight.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
