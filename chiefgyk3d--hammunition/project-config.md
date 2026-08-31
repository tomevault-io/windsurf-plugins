---
trigger: always_on
description: > Pick your RF arsenal.
---

# Hammunition — Claude Code Context

> Pick your RF arsenal.

## What this project is

Hammunition turns an existing Debian-family install into an amateur radio, SDR,
and RF experimentation workstation. Primary target: **Parrot OS**. Secondary:
Debian, Ubuntu, Kali, Raspberry Pi OS.

Binary: `hammunition`. Python package: `hammunition`.

## Document authority

`docs/DECISIONS.md` is authoritative. Where this file or `docs/DESIGN.md`
disagrees with it, DECISIONS wins and the disagreeing file is a bug.

- `docs/SCOPE.md` — the five-source union and 1.0 staging (**D-017**)
- `docs/PARITY-POLICY.md` — per-unit disposition and M5 exit criteria
- `docs/QUESTIONS.md` — decisions awaiting the maintainer, with recommendations
- `docs/reference/cli.md` — the CLI: verbs, flags, exit codes, what it refuses
- `docs/reference/source-build-gaps.md` — what the source backend cannot yet
  build, each gap named by the unit whose build proved it
- `docs/reference/capability-matrix.md` — every manifest against every target,
  resolution merged with a measured `apt-cache policy` sweep
- `docs/reference/parity-coverage.md` — every dispositioned unit against the
  catalog: what is covered, what is outstanding, and why each gap is open
- `docs/reference/` — the measurements everything rests on: `ahrl-inventory.md`,
  `blend-inventory.md`, `dispositions.md`, `overlaps.md`, `profile-sizing.md`,
  `licence-verification.md`, `hardware-gaps.md`, `udev-inventory.md`,
  `usb-ambiguity.md`, `lora-inventory.md`, `device-naming.md`

## What this project is NOT

Do not propose or build any of these. They have been considered and rejected:

- A Linux distribution, custom ISO, or derivative
- A custom kernel
- A mirror of upstream Debian packages
- Forks of upstream ham/SDR software
- Anything that replaces or reconfigures the user's OS wholesale

We **augment** an existing system. Upstream packages are used wherever they exist.

## Inventory sources

Two projects seed the catalog. Both are inventory sources; neither is a base we
build on. See **D-001** and **D-011** for the provenance rules, and credit both
in the README.

### Andy's Ham Radio Linux (AHRL)

AHRL by Andy Stewart (KB1OIQ) is the direct inspiration and the closest existing
thing to what we are building. Study it before designing anything. Treat it with
respect — it has served the ham community for well over a decade and its package
curation represents years of accumulated judgment we would be foolish to discard.

**What AHRL is (as of v27, May 2026):** no longer a distribution. It is an
installation script layered onto Debian Live, Raspberry Pi OS, or a supported
Ubuntu flavor. Supported and tested targets are Ubuntu/Xubuntu/Kubuntu 26.04,
Linux Mint 22.3, Debian Live 13.4, LMDE 7, and Raspberry Pi OS 6.2. Distributed
as versioned tarballs on SourceForge.

**What we take from it:**

- The package curation itself. Andy's selection of ham software — what is worth
  installing, what actually works, what is abandonware — is the single most
  valuable artifact in this space. Use the AHRL package inventory as the
  reference when seeding `catalog/packages/`.
- The layered-onto-existing-OS model. AHRL arrived at this after years as a
  distribution. That migration is strong evidence the augmentation approach is
  correct, and it is why building a distro is on our rejected list.
- Hard-won operational knowledge, e.g. Andy's guidance to prefer X11 over Wayland
  where ham applications misbehave. Capture that kind of thing in manifests and
  docs rather than rediscovering it in the field.

**What we do differently, and why the project exists:**

| AHRL | Hammunition |
|---|---|
| Tarballs on SourceForge | Git, tagged releases, signed |
| Single maintainer, bus factor of one | Multiple maintainers from day one, documented governance |
| Contribution by emailing the maintainer | Pull requests, issues, public review |
| Install logic and package list intertwined in shell | Declarative catalog separated from engine |
| Bash installation script | Python engine, idempotent, dry-run, transaction log |
| No automated cross-distro testing | CI containers per target distro |
| Ham radio only | Ham radio plus SDR, SIGINT, and RF security on a security-tooling base |

### 73Linux (KM4ACK)

73Linux, by Jason Oleham (KM4ACK), grew out of Build-a-Pi. Same shape as AHRL and
as us: an installer layered onto an existing Debian-family OS, not a distribution.
Actively maintained, 47 unique units across `app/stable/pi/` and
`app/stable/x86_64/`.

**What we take:** the inventory delta. 73Linux covers Winlink, packet, and EMCOMM
— PAT, PATMENU3, BPQ, AX.25, ARDOP, ARDOPGUI, VARA, GARIM, VARIM — a domain AHRL
does not touch at all. The packet core lands in 1.0 (**D-008**). Its community
side-loading model also informs our three-tier catalog (**D-009**).

**What we do not take:** any code. There is no LICENSE or COPYING in the
repository and no header on `73.sh`, so default copyright applies. A `.bapp` is
also executable bash with a metadata header — five easy fields declarative, every
hard field trapped inside an imperative `INSTALL()` body. That is the architecture
we exist to replace. See **D-001**.

**Positioning.** We are not competing with AHRL or 73Linux and must not present

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChiefGyk3D/Hammunition](https://github.com/ChiefGyk3D/Hammunition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
