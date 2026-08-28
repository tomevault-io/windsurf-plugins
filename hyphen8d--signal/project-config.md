---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

SIGNAL — a CRT/terminal internet-radio web toy. A tuning-dial receiver rendered
entirely through a text grid, playing real YouTube tracks from 9 curated
stations. Read `README.md` first: it carries the product intent, the controls
reference, and the content-ops rules that constrain what may be added.

## Commands

No build step, no dependencies. `package.json` exists only to mark the repo
`"type": "module"` (so Node runs `stations.js`, `tools/`, `tests/` as ESM) and
to name these scripts:

```bash
node tools/admin-server.mjs         # npm run admin — the admin backend + the app, port 8080
python3 tools/dev-server.py 8000    # dev server (no-store headers); open http://localhost:8000
node --test tests/*.test.mjs        # npm test — headless suite, ~2s, no network
node tools/lint-roster.js           # npm run lint — offline roster rules
node tools/verify-roster.js         # npm run verify — lint + oEmbed check of every track (network)
node tools/stations-to-md.js        # npm run stations — regenerate stations.md (never hand-edit it)
node tools/stamp.js                 # npm run stamp — bump build.json; RUN BEFORE EVERY DEPLOY
node tools/audition.js --station=<id> # npm run audition — vet candidate tracks (network)
node tools/shoot.mjs                # npm run shoot — regenerate screenshots/ (headless Chrome + ImageMagick)
node tools/dead-feedback.mjs        # npm run deadfeedback — input-feedback sweep (headless, ~1min)
```

`file://` does not work — the BDF font fetch needs a real origin. Use
`tools/dev-server.py`, not `python3 -m http.server`: only the former sends
`Cache-Control: no-store`.

**Deploys need `node tools/stamp.js`.** `main.js` fetches `build.json`
(always fresh, `?t=`) and imports every app module as `?v=<stamp>`; without a
bump, GitHub Pages' 10-minute cache can keep a visitor on the previous build.

## The admin backend

`node tools/admin-server.mjs` (`npm run admin`) serves the app AND the
network-ops dashboard at `http://127.0.0.1:8080/admin`. It sends the same
`Cache-Control: no-store` headers `tools/dev-server.py` does, so for an admin
session it replaces that server rather than running beside it. Zero
dependencies; binds loopback, checks the `Host` header, and requires an
`X-Signal-Admin` header on every mutating route — which a cross-origin page
cannot send without a CORS preflight this server never answers. That last
guard is not decorative: this process can run `git push`.

**This box is normally worked on over SSH, where `127.0.0.1:8080` names the
laptop, not this machine** — which is exactly how the first live check of the
dashboard failed, with a browser error page against a server `curl` could
reach fine. Loopback is still the default, because `tools/dev-server.py`
binding every interface is a different risk from this one doing it: that
serves static files, this commits and pushes. Two ways across:

- **Tunnel** (nothing exposed): `ssh -N -L 8080:127.0.0.1:8080 <user>@<box>`,
  then open `http://127.0.0.1:8080/admin` on the laptop. In a live session,
  `~C` then `-L 8080:127.0.0.1:8080`. The banner prints this command, filled
  in with the real port and address, whenever `SSH_CONNECTION` is set.
- **`--host=<addr>`** (`npm run admin -- --host=100.x.y.z`) binds an
  interface the other machine can see; a Tailscale address is the defensible
  one, since that is an authenticated mesh rather than the open LAN. It
  prints a warning saying what it just allowed, and there is no password.

Either way the `Host` allowlist stays on: every address this machine actually
has, plus the loopback names, and nothing else. That still stops DNS
rebinding, which needs the browser to send `Host: evil.com` — a hostname an
attacker controls is not in the set however it resolves.

`tools/network.html` was serverless until 2026-08-27, reading and writing
`stations.js` through Chrome's File System Access API with the roster parser
copied inline. **Opening the file directly no longer works** — a `file://`
page cannot import a sibling module, and re-inlining the parser is the exact
duplication that broke this dashboard once before. Its connect screen says
so and prints the command.

What the served version does that the file could not:

- **Run the Node toolchain** and stream it back: lint, the suite, verify,
  stamp, stations.md, the dead-feedback sweep, screenshots. PREFLIGHT chains
  lint → suite (roster verify is opt-in — it is the slow, networked one).
- **Edit a station's IDENTITY** — `crt`, `meter`, `ident`, `glyph`, `visual`,
  `gain`, `static`, `freq`, tagline, desc. Nothing but a hand-edit of
  `stations.js` could touch any of these before. Ident tones play through the
  same chain `playIdent()` uses; the tagline counter and the glyph-in-font
  check are the rules `lint-roster.js` already enforces, read from the server
  rather than restated here.
- **SHIP**: stamp → stations.md → lint → suite → add → commit → push,
  stopping at the first failure with nothing committed. The stamp step is the
  one most easily forgotten by hand, which is the whole argument for it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyphen8d/signal](https://github.com/hyphen8d/signal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
