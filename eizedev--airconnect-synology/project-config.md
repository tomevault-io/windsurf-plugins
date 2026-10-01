---
trigger: always_on
description: This repository and its sibling
---

# Packaging conventions

This repository and its sibling
[SpotConnect-Synology](https://github.com/eizedev/SpotConnect-Synology) package two
different upstream projects, but they solve the same problem in the same way: wrap
pre-built binaries from a [philippe44](https://github.com/philippe44) project into
Synology `.spk` packages, for a wide range of DSM devices, without building the binaries
themselves.

Both are maintained by the same person, so the same bug tends to exist in both, and a
fix found on real hardware for one is usually a fix for the other. Keeping the two
repositories structurally alike is what makes that transfer cheap: the SRM process-lookup
fix, the architecture matrix and the Package Center changelog text all started in one
repository and moved to the other more or less unchanged.

**The rule is "the same unless there is a documented reason to differ", not "identical".**
The two packages genuinely differ in places - SpotConnect stores reusable Spotify
credentials, AirConnect still supports a DSM 5/6 device line, and so on. Those
differences are deliberate and written down (see
[Deliberate differences](#deliberate-differences)), rather than left for a reader to
discover by diffing two repositories.

This repository is the reference for the shared parts. It is the older of the two, and
most of the shared conventions were arrived at here first.

## What is kept the same

### Repository layout

```text
src/dsm7/          package sources: INFO, Makefile, scripts/, conf/, WIZARD_UIFILES/, icons
tests/             validate_spk.sh, validate_elf.py, README.md explaining both
doc/               ARCHITECTURES.md, BUILD.md, CONFIG.md, TROUBLESHOOTING.md
upstream.json      the pinned upstream release
CHANGELOG.md       packaging changes (upstream has its own)
.github/workflows/ release, upstream check, linting, security scan, housekeeping
```

### Build

- `make build` builds one architecture, `make build-all` (via `build.sh`) builds every
  architecture the `Makefile` defines. Nothing else hardcodes the architecture list -
  `build.sh` and CI derive it from the `Makefile` targets, because a second copy of that
  list has silently gone stale before.
- `INFO` is a template. `#VERSION#`, `#INFO_ARCH#` and `#INFO_FIRMWARE#` are substituted
  per architecture at build time.
- Values that `INFO` already holds are not repeated in the `Makefile`: the package name,
  the packaging repository URL and the upstream repository URL are read from `package`,
  `distributor_url` and `maintainer_url`. More generally, names, URLs and versions
  belong in one place and are derived from there.
- The bundled upstream `LICENSE` and `CHANGELOG` are fetched from the pinned upstream
  tag, never from upstream's default branch, so what ships in the package matches the
  binaries that ship with it.

### Updates

An update has to keep working for an installation that was set up by a much older
release. Most installations in the field are years behind, so this is the normal case,
not an edge case.

- **Work out what an installation is actually doing, from the device.** A setting
  introduced later has no line in an older config, and an absent line says nothing about
  what that installation wants - so it must never be read as "off" or as any other
  default. Look at what is on disk instead: the links in the shared folder, the files a
  feature creates. An environment variable is not a substitute either; an older package
  cannot guarantee one is set.
- **Where a wizard cannot determine the current state, it does not offer the choice.**
  A checkbox that defaults to off is a decision the person never made, and
  `postupgrade` cannot tell that apart from a real answer. Leaving the checkbox out means
  nothing is submitted for it and the existing setting stands.
- **Where an update cannot carry the settings over at all, it refuses** with a message
  saying to uninstall and install again, rather than continuing with invented ones.
- **Only `start` may need the settings.** DSM calls `start-stop-status status` every few
  seconds and `stop` before every update and uninstall, in whatever state the installation
  is. Both answer from the processes actually running, found by their path in the package
  directory, never by bare name. Otherwise Package Center shows "stopped" while the
  binaries run, and nothing can stop them. DSM continues an update even when `stop`
  fails.
- `tests/upgrade_state.sh` and `tests/start_stop_status.sh` run these paths against
  prepared installations, so this is tested rather than assumed.

**Before releasing anything that touches installation state** - a lifecycle script, a
wizard, the config format, the shared folder, file locations or permissions - go through
this. A released package updates thousands of installations by itself, and most of them
were set up years ago.

1. Name the oldest release someone could be updating from, and open its `postinst` and
   `postupgrade`. What is in that installation's config, and what is not?
2. For every setting the new package reads: what happens when the line is missing? If the
   answer is a default rather than "find out from the device", it is wrong.
3. For every wizard checkbox on an update: what does its default do to an installation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eizedev/AirConnect-Synology](https://github.com/eizedev/AirConnect-Synology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
