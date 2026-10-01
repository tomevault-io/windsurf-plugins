---
trigger: always_on
description: Read this before continuing work. Project path: `/home/markus/Programmierung/calf/calfnxt`
---

# calfNXT — Agent handoff

Read this before continuing work. Project path: `/home/markus/Programmierung/calf/calfnxt`
(renamed from `calf_next` on 2026-07-25). Chat history may not follow the rename in Cursor.

Also see `ARCHITECTURE.md` for the high-level stack, and `VERSIONING.md`
for suite SemVer / release rules (`tools/release.sh`).

---

## What this project is

Greenfield **VST3 + WebKitGTK** (Linux first) plugin suite. One `.vst3` per plugin.
Shared React SPA UI embedded into each bundle’s `Resources/`. Branding: **calfNXT**
(namespace `calfNXT`, cmake `calfnxt`, URI `calfnxt://`, bridge `calfnxtNative`).

**Editor process model:** the VST3 `.so` must **not** link GTK/WebKit (Ardour’s
internalized toolkit collides with system GTK3 — `GdkDisplay` GType abort).
`webkit2gtk-4.1` is WebKit2 **for GTK 3**, not GTK 2. `WebEditor` in the host
process is a thin proxy: it spawns `calfnxt-web-host` (GtkPlug + WebKit, XEmbed
into the host XID) and forwards the JSON bridge over a Unix socketpair. The
helper’s spawn `envp` omits `LD_LIBRARY_PATH` (Mixbus/Ardour bundled glib breaks
system WebKit); the **host `environ` is never mutated**. Opt out with
`CALFNXT_KEEP_HOST_LDPATH`. Each bundle ships `Contents/<arch>/calfnxt-web-host`
next to the `.so`. User-facing contract: `README.md` → Clarifications.

Plugins today: **Equalizer** (`#equalizer`), **Stereo** (`#stereo`), **Transients** (`#transients`), **Compressor** (`#compressor`), **Expander** (`#expander`), **DeEsser** (`#deesser`), **Delay** (`#delay`), **Reverb** (`#reverb`), **Impulse** (`#impulse`), **Multiband Compressor** (`#mbcomp`), **Limiter** (`#limiter`), **Multiband Limiter** (`#mblimiter`), **Harmonics** (`#harmonics`), **Analyzer** (`#analyzer`), **Filter** (`#filter`), **Ring Modulator** (`#ringmod`), **Pulsator** (`#pulsator`), **Crusher** (`#crusher`), **Phaser** (`#phaser`), **Flanger** (`#flanger`), **Chorus** (`#chorus`), **Split** (`#split`), **Tuner** (`#tuner`), **Octaver** (`#octaver`), **Bender** (`#bender`), **Tamer** (`#tamer`).
Suite focus is this set — no near-term new plugins unless explicitly requested.

---

## Naming map (do not reintroduce old names)

Old brand spelling `CalfNXT` is obsolete — use **`calfNXT`**. Also never bring back
`Calf Next`, `CalfNext`, `calf-next`, `calf_next`, `calfNative`.

| Kind                   | Value                                                                                                                                                                                                                                                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Display / vendor       | `calfNXT`                                                                                                                                                                                                                                                                                                           |
| Vendor URL / email     | `https://calfnxt.org`, `mailto:schmidt@boomshop.net`                                                                                                                                                                                                                                                                |
| C++ namespace          | `calfNXT`                                                                                                                                                                                                                                                                                                           |
| CMake project / libs   | `calfnxt`, `calfnxt_ui`, `calfnxt_dsp`, `calfnxt_web_ui`                                                                                                                                                                                                                                                            |
| Plugin targets         | `calfnxt-equalizer`, … `calfnxt-bender`, `calfnxt-impulse` |
| VST3 package / `.so`   | `calfNXTEqualizer`, … `calfNXTBender`, `calfNXTImpulse` (must match; Carla/JUCE) |
| Install names          | `~/.vst3/calfNXTEqualizer.vst3`, … `calfNXTBender.vst3`, `calfNXTImpulse.vst3` |
| URI scheme             | `calfnxt://bundle/...`                                                                                                                                                                                                                                                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boomshop/calfnxt](https://github.com/boomshop/calfnxt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
