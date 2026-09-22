---
trigger: always_on
description: The example fleet for tinyjs (../tinyjsapp or github.com/tarwin/tinyjsapp).
---

# tinyjsapp-examples — working notes

The example fleet for tinyjs (../tinyjsapp or github.com/tarwin/tinyjsapp).
Each app is a folder with `tinyjs.json`; `tinyjs dev` inside it runs it.

## Releasing builds

ALL payload (mac dmg + update zip, win zip, linux tarballs) lives on GitHub
Releases — one release per app, tag `<dir>-v<version>` (version-agnostic per
release event: platforms at different versions share the tag). Only the
small stuff (manifests, catalog, README) is committed. Payloads were PURGED
from git history 2026-07-25 — never commit one again; `_builds/` is a local
staging area, gitignored except `_builds/<dir>/manifest.json`, which shipped
apps poll by raw url — never remove or purge the manifests.

⚠ History was force-rewritten for that purge (683 MB → 20 MB). Any clone
from before 2026-07-25 must `git fetch && git reset --hard origin/main &&
git fetch --tags --force` — never pull/merge across the rewrite, that
resurrects the old 683 MB history.

How auto-update works: each shipped app polls its baked-in raw url
`_builds/<dir>/manifest.json`, reads its own platform block (top level =
mac, `win`, `linux.<arch>`), and downloads that block's `url` verifying
`sha256`. The manifest is the mutable pointer; release assets are the
static payload. So a release = upload assets to the tag, then push updated
manifest/catalog urls. Uploading needs `gh` authed with repo scope.

### macOS / Windows

⚠ Windows: run `tinyjs publish` from PowerShell/cmd, NOT Git Bash. The
zip is written by `tar -a -cf` and Git Bash’s GNU tar shadows Windows’
bsdtar in PATH — GNU tar has no zip writer, so it silently emits a TAR
named `.zip` that Explorer/Expand-Archive open as empty (bsdtar still
reads it, so the in-app updater survives; human downloads do not).
Check before uploading: `Expand-Archive` must yield 3 entries.

1. Bump `version` in `tinyjs.json`, build + publish as usual, stage the
   artifacts where they always went: `_builds/<name>-<ver>.dmg` (mac
   human download), `_builds/<dir>/<name>-<ver>.zip` (mac update payload),
   `_builds/<dir>/<name>-<ver>-win.zip`. They stay local (gitignored).
2. `gh release create <dir>-v<ver> -R tarwin/tinyjsapp-examples` if the tag
   is new, then `gh release upload <dir>-v<ver> <files> --clobber`.
3. Edit `_builds/<dir>/manifest.json` — ONLY your platform's block (top
   level for mac, `win` for windows): version, sha256, notes, and url =
   `https://github.com/tarwin/tinyjsapp-examples/releases/download/<tag>/<file>`.
   Never copy a published manifest over it wholesale (kills other platforms).
4. Mac only: `node shelf/gen-catalog.js` (mac-only: sips; `EXAMPLES_ROOT`
   env overrides its hard-coded repo path). It rebuilds mac entries and
   carries win/linux/platforms blocks over from the existing catalog.json;
   apps without a staged dmg for the current version keep their old entry.
   Windows: no gen tool exists — edit the catalog `win` blocks in
   catalog.json AND shelf/src/frontend/catalog.js by hand (keep in sync),
   or model a merge tool on merge-manifest-linux.js.
5. Update the download line in README.md (+ the app's own README for mac).
6. Verify urls (`curl -fsSLI`), commit manifests + catalog + README, push.

### Linux — normally CI: the `linux-release` workflow

Default path, from any machine (no docker, no Linux VM):

```
git push                                   # CI builds what is COMMITTED
gh workflow run linux-release.yml -R tarwin/tinyjsapp-examples \
  -f tinyjs_tag=v0.36.0 -f apps="shelf nib"   # apps empty = fleet minus amp
gh run watch <id> --exit-status ; git pull --rebase origin main
```

Manual dispatch only. It runs steps 2–7 below (bar the docs site) — same
`pkg-linux-container.sh` in an ubuntu:22.04 container on both arches
(x86_64 on `ubuntu-latest`, arm64 on `ubuntu-24.04-arm`, ~2 min total),
uploads to the per-app release tags, merges both arch passes into the
committed manifests, regenerates the catalog, bumps README's Linux links,
and pushes. Verified on shelf 0.2.8 (2026-08-01).

Left for you afterwards: ../tinyjsapp/docs/index.html's Linux links (other
repo), the mac/win halves of the release, and a `git pull --rebase` before
your own manifest edits — the workflow pushes to main. Bump each app's
`tinyjs.json` version and commit BEFORE dispatching. An app absent from
catalog.json (shelf) is skipped by gen-catalog with a note; that is fine.

### Linux by hand (the local-container fallback — no CI, or debugging it)

⚠ NEVER `tinyjs publish` for Linux on the host. The linker bakes the build
userspace's glibc floor into tjs + launcher, and this VM is Ubuntu 24.04 —
tarballs packaged here demand GLIBC_2.38 and refuse to load on Ubuntu 22.04
/ Debian 12 / Mint 21 (`version GLIBC_2.38 not found`). That is exactly how
amp 0.8.0–0.10.0 and the whole 2026-07 fleet shipped broken. Everything
builds inside `ubuntu:22.04` containers; Rosetta (enabled on this VM,
`/proc/sys/fs/binfmt_misc/RosettaLinux`) runs the amd64 one.

1. Bump `version` in each changed app's `tinyjs.json`.
2. Run `shelf/pkg-linux-container.sh` once per arch (usage in its header:
   two `docker run` lines, `-e TINYJS_TAG=<tinyjs release>`). It builds the
   toolchain from the tagged tinyjs release inside 22.04, publishes each

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarwin/tinyjsapp-examples](https://github.com/tarwin/tinyjsapp-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
