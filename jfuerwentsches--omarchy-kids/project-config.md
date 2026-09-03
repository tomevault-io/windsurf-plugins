---
trigger: always_on
description: A configuration layer on top of [Omarchy](https://omarchy.org) that grows
---

# omarchy-kids — working notes for Claude Code

A configuration layer on top of [Omarchy](https://omarchy.org) that grows
with a child — age-tiered desktop profiles plus tooling for parental
controls and screen time. Not a fork: Omarchy installs normally, this
project layers config, a background agent, and a control center on top.

## Where the actual design lives

The maintainer keeps the concept/design source of truth in a private notes
system outside this repo — not published here, and not needed to build or
run anything in it. If you're the maintainer (or working from a session
that has access to it), it's linked from a local, gitignored
`.claude/CLAUDE.local.md` — start there for design rationale before making
architectural decisions. Everyone else: this file, the folder-level
READMEs, and commit history are the available context.

For orientation, the private notes are organized around these topics (contributors without access can safely ignore this list — it's here so issues/PRs can reference the right area by name):

- `Omarchy Kids (mitwachsendes OS)` — hub note, links everything
- `Omarchy Kids - Architekturuebersicht` — full component map + security principles
- `Omarchy Kids - Altersstufe 5-7` / `- 8-10` — per-tier concept (apps, UI principle)
- `Omarchy Kids - Implementierung Agent` — agent/agentd design + why the split matters
- `Omarchy Kids - Implementierung Control Center` — control center design
- `Omarchy Kids - Implementierung Launcher` — kiosk launcher plugin, the Quickshell gotchas
- `Omarchy Kids - Themes` — per-tier theming, franchise-licensing stance
- `Omarchy Kids - Sprache und Locales` — German-from-day-one i18n plan
- `Omarchy Kids - Entwicklungsumgebung` — dev environment design rationale
- `Omarchy Kids - Open-Source-Struktur und Paketierung` — repo/license/packaging decisions

## Current focus

**Only tier "mini" (age 5-7) is being built right now** — explicitly
deprioritized the other tiers (midi/8-10, maxi/11-13, teen/14-16) until mini
is solid. Don't start scaffolding other tiers unless asked.

## Status (as of the last session)

- Monorepo scaffold in place: `agent/` (Rust workspace, builds), `control/`
  (CMake/Qt6, builds), `tiers/`, `quickshell-plugin/`, `setup-wizard/`,
  `docs/` — see each folder's README for stack/status.
- `tiers/mini/` is a working end-to-end kiosk, verified in the dev VM:
  - `themes/` — a tier can ship multiple themes now (own artwork, no
    licensed franchises): `sternenreise` (default, space) and
    `meerjungfrauen` (underwater/mermaid). `omarchy-kids-set-tier <tier>
    [theme]` installs all of a tier's themes plus any dropped into
    `~/.config/omarchy-kids/themes/<tier>/` (same folder shape, no code
    change needed), and activates the given theme or
    `themes/default-theme.txt`.
  - `launcher/omarchy-kids.launcher/` — fullscreen Quickshell overlay plugin,
    icon-only grid, launches via `gtk-launch`. App line-up is being
    reworked (see the Altersstufe-5-7 vault note): GCompris is being forked
    as `omarkid-gcompris` for theming, KTuberling stays as-is, Tux
    Paint/Klettres/Blinken were dropped entirely — `apps.json` in this repo
    still reflects the old line-up until that lands.
  - `hypr/hyprland.lua` — full Hyprland config replacement: every default
    binding off, only `SUPER+SPACE` survives (→ the kiosk launcher)
  - `omarchy-kids-set-tier` — applies all of the above, plus masks
    `getty@tty2-6` (VT-switch lockdown) for this tier
- `agent/` (issues #1-#15, plus #10 and #25 from the other two project
  boards — see below) is implemented, not a stub: `agent`/`agentd`/
  `omarchy-kids-run`/`omarchy-kids-override-helper`/
  `omarchy-kids-repair-helper` — protocol, time budgets, PIN override,
  production re-pairing trigger, packaging. See `docs/agent-protocol.md`.
  Found and fixed along the way (2026-08-29): the pairing-installed
  `authorized_keys` entry's `command=` restriction execs `omarchy-kids-agent`
  with zero argv, ignoring `$SSH_ORIGINAL_COMMAND` — every remote command
  (not just Control Center's new dashboard below) silently collapsed to a
  usage error until `agent/src/main.rs` gained `effective_args()` to read it
  back out. Prior pairing verification never caught this because it only
  ever tested the SSH *login*, never a real remote command.
- `setup-wizard/` (issues #16-#27 done so far, project board:
  [Pairing & Setup Wizard](https://github.com/users/jfuerwentsches/projects/3))
  also has real logic now, not just a stub:
  - `setup-wizard/bootstrap/` — Phase 1 scripted bootstrap (account/wheel
    topology, branding, initial tier switch), verified end-to-end in the
    dev VM. See its README.
  - `agent/pairing/` (`omarchy-kids-pairing` binary, lives in the agent
    workspace though it's the setup-wizard's Pairing track) — child-to-
    Control-Center pairing exchange (mDNS + QR discovery, SPAKE2-
    authenticated key handoff), verified over a real network in the dev
    VM. See `docs/agent-protocol.md`'s "Pairing protocol" section.
  - `setup-wizard/first-boot/` — the first-boot hook (issue #27): a
    systemd unit chained `After=omarchy-provision-owner.service`,
    `Before=display-manager.service` (Omarchy 4 has no official first-boot

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jfuerwentsches/omarchy-kids](https://github.com/jfuerwentsches/omarchy-kids) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
