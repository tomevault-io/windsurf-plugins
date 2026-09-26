---
trigger: always_on
description: provides only the minimal bootstrap toolchain (gcc, binutils, make, etc.);
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this
repository. It contains rules that shape behavior; project overview and
architecture description live under `docs/` (start with `docs/intro.md` and
`docs/architecture.md`).

## Key Design Decisions

- **No installing packages in the container.** If a build fails because a tool
  or library is missing from the container, the fix is to write a unit that
  builds it from source (and add it as a `deps` entry), not to install it via
  `apk add` in the Dockerfile. This applies to both build tools (makeinfo,
  bison, help2man) and libraries (zlib, ncurses, libffi). The Dockerfile
  provides only the minimal bootstrap toolchain (gcc, binutils, make, etc.);
  everything else is a unit. For non-essential features (docs, man pages),
  disabling via configure flags is also acceptable.
- **Need a tool Alpine already packages? Pull it through `module-alpine`, don't
  build from source.** See the `pulling-alpine-packages` skill for the workflow,
  the `module-alpine` cache layout, and the push-upstream rules.
- **Container units set arch explicitly.** Classes set `container` and
  `container_arch` explicitly; units inherit these from their class. Do not let
  container selection happen by implicit default.
- **Prefer git sources over tarballs.** Shallow clone with tag pinning. Enables
  the `yoe dev` workflow (edit, commit, extract patches). The exception is a
  host that cannot be relied on to answer — `git.savannah.gnu.org` stalls
  mid-clone often enough to fail CI — where a release tarball with a `sha256` is
  the better source, since the project's `source_mirrors` table gives it a
  fallback host and a git source has none. See the `new-unit` skill.
- **Native builds only — no cross-compilation.** Cross-arch is handled by
  foreign-arch containers under QEMU user-mode (binfmt_misc); never propose a
  cross-compile toolchain instead.
- **Content-addressed caching.** Input hash determines output. When adding a new
  unit field that participates in the hash (`internal/resolve/hash.go`), gate
  the `fmt.Fprintf` on a non-empty/non-zero check so units that don't set the
  field stay cache-neutral. An unconditional write invalidates every unit's hash
  the moment the line lands, forcing a full rebuild. Follow the pattern used for
  `Extra` and the image-only block.
- **Hardware-bootable images.** Images must boot on real hardware, not just
  QEMU. Never suggest QEMU-only shortcuts like `-kernel` direct boot that bypass
  the bootloader. QEMU is a development convenience; the real target is always
  physical boards.
- **No intermediate code generation.** Avoid generating shell scripts, config
  files, or other intermediate artifacts that are then executed or parsed. When
  something fails, the user should be looking at the code they wrote, not
  machine-generated output. Prefer direct execution over code generation.
- **One unit is the single definition; reuse binaries, resolve variation at
  runtime.** A unit is the one place a package's build is defined. Its output is
  content-addressed and keyed by `(distro/libc, arch, machine scope)`: every
  project and machine that lands on the same key shares the same artifact. The
  same source unit therefore builds more than once whenever those keys differ —
  most commonly along the distro axis, where musl-built and glibc-built binaries
  cannot share at the ABI level, so a unit consumed by both Alpine and Debian
  images builds twice. That fan-out is normal, not an exception. A unit may also
  emit more than one package artifact where split packages (e.g. `-dev`, `-doc`,
  `-libs`) apply; the unit stays the single definition while its outputs fan
  out. So "one .apk" is the common case, not an invariant — the invariant is
  that the unit is the single source of truth for the build. Within a given key,
  the goal is still one shared artifact rather than per-project or per-machine
  forks. Note what this rule is _not_ about: varying along an axis the key
  already splits on. A build step that branches on `$DISTRO` or `$ARCH` — both
  exported to every step, both already unit-hash inputs — forks nothing, because
  those artifacts exist separately either way; the branch only decides what goes
  into each. There is no reuse to weigh and nothing to deliberate. Prefer it to
  shipping every variant's files into every artifact and leaving the running
  system to ignore what does not apply — that is dead weight, not runtime
  resolution. Service files are the worked example: the OpenRC script belongs in
  the Alpine package and the systemd unit in the Debian one, gated on `$DISTRO`,
  rather than both landing in both. What follows is about the other case —
  variation the key does _not_ capture, where two projects or two machines want
  different behavior from one artifact. There, reach for runtime mechanisms
  first — init scripts that detect what's installed, conditional config files,
  alternative selection at boot, `replaces:` annotations that declare ownership
  of shared paths — before forking build configuration. Forking a unit's build
  flags into machine- or project-scoped variants is the most expensive option:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoebuild/yoe](https://github.com/yoebuild/yoe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
