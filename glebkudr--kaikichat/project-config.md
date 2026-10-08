---
trigger: always_on
description: **Machine-specific rules first.** If `AGENTS.local.md` exists in the
---

# Agent guide

**Machine-specific rules first.** If `AGENTS.local.md` exists in the
repository root, read it before any build, check, release or deployment. It
is not part of Git (see `.gitignore`) and names this machine's paths, disks,
toolchains, keys, servers and what the owner allows without asking; where it
is more specific than this file, it wins. Never commit it, and never copy its
contents into tracked files.

**Never delete recursively outside the working directory.** No `rm -rf`
outside the repository checkout or the worktree the task runs in: not in
`/tmp`, the home directory, other checkouts or other volumes. The same holds
for any other recursive deletion (`rm -r`, `find … -delete`,
`shutil.rmtree` and the like). Leave temporary files outside the working
directory in place, or ask the owner. State this rule in the task of every
subagent you start.

# Storage and builds

Every build and check goes through `scripts/build-storage.py`: it checks where
the heavy build data lives, sets the tool and cache paths and starts the
command. Pick its profile explicitly: `mac-apfs` (the default) on macOS,
`portable-linux` on Linux.

## The mac-apfs profile

The heavy build directories (see Placement rules) are symlinks into a
dedicated APFS volume attached from a disk image. `.local/build-storage.json`
(not in Git) describes it:

- `image`, `mount`, `volume_uuid`: the sparse bundle, where it is attached
  and the UUID the volume must have;
- `directory` (optional): this checkout's directory on the volume, when
  several checkouts share one image; the volume's root otherwise;
- `path`, `env`: the project's toolchains put first on `PATH`, and variables
  such as `AIN_NODE`, `AIN_FOUNDRY_BIN`, `AIN_SOLC`.

## Commands

Run from the repository root.

```sh
python3 scripts/build-storage.py mount          # Attach the image without starting a build
python3 scripts/build-storage.py check          # Verify the volume and symlinks without attaching anything
python3 scripts/build-storage.py setup          # Restore empty directories and missing symlinks
python3 scripts/build-storage.py install        # Restore npm from the lock file inside the image
python3 scripts/build-storage.py run node scripts/build-desktop.mjs
python3 scripts/build-storage.py run scripts/check.sh
python3 scripts/build-storage.py run node scripts/check-native.mjs
python3 scripts/build-storage.py run node scripts/check-network.mjs
```

The same format applies to individual commands:
`python3 scripts/build-storage.py run COMMAND...`.
`run` attaches the image automatically; there is no need to run `mount`
separately each time. The built release app:
`target/release/bundle/macos/Kaiki Chat.app`.

### Explicit portable profile for Linux

On Linux work in the root of the current checkout and always choose the
profile explicitly:

```sh
python3 scripts/build-storage.py --profile portable-linux setup
python3 scripts/build-storage.py --profile portable-linux doctor --suite build-tooling
python3 scripts/build-storage.py --profile portable-linux run python3 -m unittest discover -s tests/build -p 'test_build_evidence.py' -v
python3 scripts/build-storage.py --profile portable-linux --output output/build-evidence-001 run python3 -m unittest discover -s tests/build -p 'test_build_evidence.py' -v
```

`portable-linux` is available only on Linux. It uses `target`, `output`,
`cache` and `.local/verification-workspaces` of the current checkout and never
calls APFS, `diskutil` or `hdiutil`. These paths and their parents inside the
checkout must be ordinary directories: any symlinks are rejected before
directories are created or anything is written, and are left untouched.
`CARGO_TARGET_DIR` must point at the target of this checkout;
`CARGO_BUILD_TARGET` and `CARGO_BUILD_TARGET_DIR` are forbidden. The profile
mounts no images and installs no dependencies. The `run` command starts the
child process from the absolute root of the checkout. With `--output` it saves
a command log and `check.json`; a new attempt needs a new directory. Without
`--output` a plain exec is recorded.

The default profile is `mac-apfs`: a missing `.local/build-storage.json`,
volume or utilities, or a UUID/APFS/image/symlink mismatch remains an error,
including for `check`. The presence of Linux or the absence of the Mac config
does not enable portable automatically. The wrapper passes the chosen profile
to the nested first-party scripts through `AIN_BUILD_STORAGE_PROFILE`; those
scripts again name `--profile` explicitly. `build-storage.py` itself does not
choose the profile from an environment variable.

Doctor checks only the selected suite and reports `passed`, `failed`,
`blocked` or `unsupported`; it is a preflight, not product acceptance. The
Python suites need no Cargo, frontend-unit needs no Tauri/Foundry,
native-macos is unavailable on Linux. native-runtime has only a runtime phase:
Python, Git and local Unix IPC for daemon scenarios, without Cargo/Foundry.
Doctor does not check `AIN_SOLC`. Doctor does not check the mac-apfs tools
outside Darwin and returns `unsupported`. `build-desktop.mjs` seals into
`output/desktop-build/<run>/prepared-artifacts.json` the sources/ref, locks,
the build configuration (including recursive Cargo `include`) and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glebkudr/kaikichat](https://github.com/glebkudr/kaikichat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
