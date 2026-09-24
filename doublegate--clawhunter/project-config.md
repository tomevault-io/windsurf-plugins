---
trigger: always_on
description: <!-- Managed by Master-Claude. Universal rules come from the imported/inlined core.
---

<!-- Managed by Master-Claude. Universal rules come from the imported/inlined core.
     Edit only inside the MC-PROJECT block; mc-sync overwrites everything else. -->
<!-- mc-core: 0.2.0 | mode=import | lang=generic -->
# AGENTS.md — CLAWHunter

@/home/parobek/.claude/master-core/AGENTS.base.md
@/home/parobek/.claude/master-core/lang/generic.md
@/home/parobek/.claude/master-core/modules/10-commits-and-versioning.md
@/home/parobek/.claude/master-core/modules/20-testing-and-accuracy.md
@/home/parobek/.claude/master-core/modules/30-quality-gates.md
@/home/parobek/.claude/master-core/modules/40-docs-and-adrs.md
@/home/parobek/.claude/master-core/modules/50-architecture-patterns.md
@/home/parobek/.claude/master-core/modules/60-security.md
@/home/parobek/.claude/master-core/modules/70-release-ceremony.md
@/home/parobek/.claude/master-core/modules/80-phase-sprint-workflow.md
@/home/parobek/.claude/master-core/modules/90-multi-language-integration.md
@/home/parobek/.claude/master-core/modules/91-agent-system-architecture.md
@/home/parobek/.claude/master-core/modules/95-named-pattern-library.md

<<< MC-PROJECT-START >>>

## Project: CLAWHunter

> Hand-authored. `mc-sync` never overwrites content between the MC-PROJECT markers.
> Per-project truth only — universal rules come from the imported core above.

> This is the **`main`** branch — the Hak5 Pager payload suite. The native **Monstatek M1** C
> port is a **separate device** and lives only on the **`M1`** branch (`vendor/`, `docs/M1-PORT.md`).
> Do not describe or reintroduce M1 specifics here.

> `CLAUDE.md` and `GEMINI.md` are **symlinks to this file**. Edit `AGENTS.md`; never
> replace either symlink with a real file or the three agents silently fork.

- **What it is:** Hak5 WiFi Pineapple **Pager** payload suite for discovering and assessing OpenClaw AI-gateway instances on authorized local networks. Three payload variants (interactive, recon-triggered, alert-fired) share one fingerprinting library, with full hardware integration (color display, RGB LEDs, haptics, audio, results browser, harvest engine).
- **Stack:** Bash (payloads + shared lib) + Python3 (`harvest.py`, stdlib-only). Runs on the Pager's OpenWRT userland; no build step — payloads execute in place.
- **Deploy:** copy payloads to the Pager; install larger deps (e.g. python3) to internal 4 GB eMMC with `-d mmc` — the root overlay has <32 MB free after firmware.
- **Test / lint:** `scripts/check.sh` is the deterministic host/release gate (Bash syntax, ShellCheck, Python compile/unit tests, loopback integration, invariants, reproducible packaging, checksums, and installer dry run). Physical Pager validation remains a separate recorded hardware gate.
- `CONTRIBUTING.md` holds the 11 numbered **architecture contracts** (regression coverage required when changed), the mandatory commenting standard, and the PR checklist. Read it before changing classification, checkpointing, parallelism, or packaging.
- CI: `.github/workflows/quality.yml` runs `scripts/check.sh` on every push/PR; `release.yml` fires on a `v*.*.*` tag, rebuilds from the tag (never local `dist/`), asserts the tag equals `package-release.sh`'s `VERSION`, and refuses to replace an already-published release.

### Architecture — load-bearing facts

- Three modes — **recon** (RF-first, auto-connect) / **user** (interactive, all features) / **alert** (auto-fires, <5s, silent watchdog) — all source one shared library, `lib/common.sh` (LED, audio, fingerprinting, harvest trigger). Add cross-cutting behavior there, not per-payload.
- Fingerprinting pipeline is IPv4 port-scan based, with mDNS + ARP-cache harvesting; IPv6 link-local (`fe80::/10`) neighbors are logged as candidates only (no port scan — scope-ID complexity).
- Scan speed is profile-driven (Ghost/Quiet/Normal/Fast/Aggressive); NORMAL/QUIET scan sequentially, FAST/AGGRESSIVE in parallel — both paths must honor checkpoint resume identically.

### Gotchas / institutional knowledge

- **A version bump touches 11 files.** Six carry runtime declarations that `check.sh` parses into one unified value (`lib/common.sh`, the three `payload.sh` files, `harvest.py`, `scripts/package-release.sh`); four are user-facing surfaces asserted as exact strings (`README.md`'s `version-3.4.0-green` badge, `CHANGELOG.md`'s `^## [v3.4.0] - <date>$` **including the date**, `scripts/install-pager.sh`, `docs/architecture.dot`); and `check.sh` itself hard-codes the expected value so an accidental *unified* downgrade still fails. Count files, not occurrences — several files hold more than one (`check.sh` has 10, `README.md` 11). Bumping only the runtime declarations fails the gate.
- **The gate is not BusyBox-constrained.** `scripts/check.sh` needs `shellcheck`, `rg` (ripgrep), `curl`, and `nc` (netcat) on the *host*. The stdlib-only / BusyBox-compatible rule governs shipped payload code, not the gate. A missing `nc` surfaces only as `FAIL: loopback classifier probe failed` — `probe_openclaw` gates on `nc -z` (`lib/common.sh:229`) — which reads like a classifier bug rather than an absent tool. CI never hits this because `ubuntu-latest` ships netcat.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doublegate/CLAWHunter](https://github.com/doublegate/CLAWHunter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
