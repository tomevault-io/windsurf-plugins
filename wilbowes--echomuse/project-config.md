---
trigger: always_on
description: The Go binary that runs on the Echo Dot. Project-wide direction, the
---

# CLAUDE.md — `device/`

The Go binary that runs on the Echo Dot. Project-wide direction, the
device/controller compatibility rules, the wire protocol and the release
scheme are in the repo-root `CLAUDE.md`; the controller half is in
`controller/CLAUDE.md`.

## Building the device binary

The Echo Dot runs FireOS 5 (API 22). Standard Go cross-compilation won't work — a custom Docker build environment is required.

**One-time setup:**
```bash
# GoTinyAlsa is a git submodule at the repo root — the wilbowes/GoTinyAlsa
# fork, NOT upstream Binozo, pinned to the fork's master. It carries two
# GetAudioStream fixes: the defer-in-loop leak (v2.9.2) and a fresh slice per
# read (fork PR #1, #607 — one reused buffer meant queued batches were
# overwritten by the next read). Don't repoint it upstream until both are
# merged there (Binozo/GoTinyAlsa#2 is the second).
git submodule update --init

# Build the compiler Docker image (from device/)
cd device
docker build -t echomuse-compiler compiler/
```

**The compiler base is pinned by DIGEST, and must stay that way.**
`compiler/Dockerfile` carries the Go toolchain (1.24.0) and NDK
(21.4.7075529) that compile the firmware, so it is the layer sitting
directly on top of FireOS 5 — a 2015 platform that cannot be upgraded. It
was `FROM ghcr.io/binozo/echogo:latest`, a third party's floating tag, and
`release.yml` rebuilds the image **from scratch on every tag push**: every
release was free to pick up a different compiler than the last, with no PR
and no CI signal. The first symptom would be a binary the hardware refuses
to run, which is the one failure here not recoverable from the dashboard.

Moving the pin needs **a real device in the loop**. The host tests and
`go vet` cannot speak to it — they run on amd64 with the host toolchain,
and this image is exercised only by `compile.sh` and `release.yml`, so a
green CI run on a pin change proves nothing about it.

**Compile:**
```bash
cd device
./compile.sh
# Output: build/server
```

`compile.sh` embeds the git version string via `-ldflags "-X .../client.Version=..."`. Dirty trees get a `YYYYMMDD-HHMM-dev` timestamp instead of the tag.

**Run Go tests (host):**
```bash
cd device
go test ./...
```

Tests only cover pure-Go logic — hardware-dependent code is not testable on the host.

**Run controller tests (host):**
```bash
cd controller
python -m pytest tests/        # needs: pytest numpy scipy pyyaml — not the full requirements.txt
```

Controller tests cover the pure-logic modules only (`em_eq`, `em_limiter`, `em_mbc`, `em_scenes`, `em_oww_models`, `em_oww_warmup`, `version`, `em_hostip`, `em_ingressauth`, and the decision modules — `em_linkauth`, `em_button`, `em_shadow`, `em_turnclock`, `em_runbarrier`, `em_announce`) — keep it that way unless you're prepared to pull openwakeword/aiohttp into the test environment. Both suites (plus `go vet`) run in CI on every push/PR (`.github/workflows/ci.yml`).

**Release:** pushing a `v*` tag triggers `.github/workflows/release.yml`, which builds the binary in the compiler image and attaches it to a GitHub release. **Tag with `git tag -a --cleanup=verbatim`** — the annotation message becomes the release body (`body_path` from `git tag -l --format='%(contents)'`), which is what the dashboard shows next to an available update. Write it for the person deciding whether to push firmware to a device they depend on: what changed, what to expect, anything required of them. GitHub's generated commit list is still appended below it. A lightweight tag yields an empty body and falls back to that list, which is a worse experience, not a broken one.

**`--cleanup=verbatim` is not optional if the notes use Markdown headings.**
`git tag -a` defaults to `--cleanup=strip`, which treats a line beginning with
`#` as a comment and deletes **the whole line** — so `## Volume` does not lose
its markers, it disappears entirely. v2.12.0 shipped that way: the notes were
structurally correct in the file, five headings gone from the published body,
and the only visible sign was a wall of paragraphs. Fixing it afterwards means
`gh api -X PATCH repos/<owner>/<repo>/releases/<id> -F body=@notes.md`
(`gh release edit` has no `--notes-file`), and re-appending GitHub's generated
commit list by hand, since the PATCH replaces the whole body.

## Device audio pipeline

Playback has a second plane: music rides `0x04`/`0x05` into its own buffer and
is mixed against voice at the ALSA write, so a voice turn **ducks** music
rather than pausing it. The rules for that mix — the constant-slew ramp, the
per-sample interpolation, `music_flush` vs `speaker_flush` — are under
"Ducking" in `controller/CLAUDE.md`.

Each mic buffer passes through, in order:

```
raw 9ch S24_3LE → beamformer + fixed mic gain (micGainDb, applied to 24-bit samples) → mono S16_LE → [AEC] → [AGC] → [VAD gate] → /data WebSocket
```

Note the real buffer cadence: GoTinyAlsa's `GetAudioStream` reads the whole ALSA buffer per chunk (PeriodSize 512 × PeriodCount 5), so the mic pipeline runs on **160ms batches of 2560 samples**, not single 32ms periods. Anything assuming 512-sample buffers must handle multiples (this silently disabled AEC for four releases — see `aec.Process`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilbowes/EchoMuse](https://github.com/wilbowes/EchoMuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
