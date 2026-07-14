---
trigger: always_on
description: Make a Claude Code session multiplayer: host CLI wraps Claude in a PTY, a relay
---

# claude-share — project context

Make a Claude Code session multiplayer: host CLI wraps Claude in a PTY, a relay
forwards bytes, guests join from a browser. See README.md for usage/architecture.

## Deployed state (2026-07-11)

- Relay live at **https://claudecollab.org** (Fly app `claudeshare`, region dfw;
  claudeshare.fly.dev still answers). Certs: apex + www, Let's Encrypt via fly
- Host command: `node packages/cli/bin/claude-share.js --relay ssh://claudecollab.org:2222`
  (ssh door rides the same A record; needs CLAUDE_SHARE_SECRET in the environment)
- Relay ssh fingerprint (pin/verify): `SHA256:K2JkcwxqpWo5F+CP5q5Ke7RGTZYgjJ7tE/ajEajxdxY`
- **Exactly ONE machine, always** (`fly scale count 1`) — rooms live in the relay
  process's memory; Fly auto-adds a second machine on fresh deploys, scale it back
- A redeploy/restart ends all live rooms (hosts drop to solo) — deploy between sessions
- Deploy with `fly deploy` from the repo root; the dashboard's GitHub deployer fails opaquely
- Dedicated IPv4 168.220.81.240 ($2/mo, required for the raw-TCP ssh door) + IPv6
- `HOST_KEY` secret set (regen: `node packages/relay/bin/serve.js --make-key`)
- ROOM_SECRET set on Fly (2026-07-11) — room creation is gated; Ian's
  CLAUDE_SHARE_SECRET lives in his ~/.zshrc
- The CLI pins the relay's ssh fingerprint on first connect
  (`~/.claude-share/known_relays.json`, TOFU; `--fingerprint` pins explicitly;
  loopback exempt). Rotating HOST_KEY makes every host refuse until they clear
  the pin — rotate deliberately

## Dev & testing

- `npm test` — full suite, keep it green
- node-pty spawn-helper exec-bit gotcha — FIXED 0.1.3 (2026-07-13). Installers
  that skip build scripts (npm allow-scripts / ignore-scripts, pnpm default) ship
  spawn-helper non-executable → "posix_spawnp failed" on first spawn. startPty()
  now self-heals at RUNTIME via `ensureSpawnHelperExecutable()` in pty.js (chmods
  every prebuilds/*/spawn-helper before spawning; a postinstall can't — same
  policies block it). The error hint stays as a last resort if chmod can't (perms).
- `scripts/rig.sh` — local relay + host wrapping a FAKE claude (test/fixtures/fake-claude.cjs),
  ports **2226 (ssh) / 8788 (web)**; `scripts/rig-real.sh` — same with the real claude
- Never use ports 2222/8787/8080 for test rigs — those are Ian's live/dev ports
- Verify UI changes by driving a real browser tab against the rig and screenshotting;
  prefer scripted ws clients / synthetic events over typing into real windows

## Conventions

- Fast direct iteration: edit → verify live → screenshot → commit. No heavy orchestration.
- Concise conventional commits. No AI attribution, no Co-Authored-By trailers.
- If something looks broken while working, say so instead of silently working around it.

## Backlog (parked, in priority order)

1. Direction DECIDED 2026-07-11: **free open-source project, not a business.**
   MIT license (LICENSE committed). Domain **claudecollab.org** = landing page
   with a GitHub link; the deliverable is "install one command, run it" (makes
   backlog #4, the Rust single binary, the headline item). No accounts, no
   payments, no hosted platform. Trademark risk accepted knowingly by Ian —
   OSS/free does NOT immunize the name (Clawdbot was free OSS and was still
   forced to rename); if traction draws enforcement, rename then. Contingency:
   keep the brand SKIN-DEEP (domain + landing copy only — never in binary names,
   protocol strings, or config paths). Pre-cleared fallback: **barge.sh**
   (available + collision-free 2026-07-11; coterm.co, ptyparty.co also clear).
   Framing rule regardless: "collaborate on a live session," never "share your
   Claude subscription."
2. DONE 2026-07-11: claudecollab.org bought (Namecheap), certs issued (apex+www),
   A/AAAA → dedicated IPs, PUBLIC_URL switched, ROOM_SECRET activated — one deploy
2a. Community relay (decided direction 2026-07-11): claudecollab.org doubles as the
   free public relay so OSS users get "install → run → share a link" with zero
   infra (the tmate/sshx/Syncthing model; terminal bytes are cheap, ~$7/mo until
   real traction). A community relay runs WITHOUT ROOM_SECRET — protection is the
   maxRooms lid + per-IP knock limits; the Rust CLI will default its --relay here.
   Prereq feature — DONE 2026-07-11: **optional per-room join password**
   (`--room-password`; HELLO `pass`; ssh masked prompt + web reveal-on-demand
   field; wrong guesses burn tryKnock slots; host tab exempt; knock/admit stays
   the final gate). Keep Ian's private secret-gated mode working from the same
   code path.
3. DONE 2026-07-11 (code + ops): relay hardening — HELLO room secret + relay-key
   pinning shipped; ROOM_SECRET live on Fly
4. Rust port — PARKED 2026-07-11 (Ian: "isn't necessary at all"). The audience
   runs Claude Code, an npm package — they have Node by definition, so npm IS the
   native install channel and the single-binary thesis dissolved. The `rust/`
   workspace + `collab-protocol` crate (spec-port of protocol.js, 7 tests green)
   stays in-repo as the restart point. Revisit only on a concrete trigger:
   real non-Node demand, relay scale pain, or a brew-distribution moment.
4a. LAUNCH PATH: ✅ npm published 2026-07-11 (**@claudecollab/cli@0.1.0**, org

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ir272/claudecollab](https://github.com/ir272/claudecollab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
