---
trigger: always_on
description: A PySide6 editor for osu!taiko charts. `gui.py` is the monolith (~12k lines);
---

# Taiko Fancy Arranger — working notes

A PySide6 editor for osu!taiko charts. `gui.py` is the monolith (~12k lines);
`osu_io/` parses and writes `.osu`, `model/` holds the undo commands,
`gimmick_session.py` builds gimmick structures, `time_axis.py` is the shared
scrolling/snapping mixin.

## Measure first. Every time.

This is the rule the project keeps re-learning, so it goes first.

Four separate investigations here reached a confident, reasonable, **wrong**
diagnosis before anyone measured:

| The confident answer | What measurement said |
| --- | --- |
| Pure-Python time-stretch is infeasible without numpy | 0.11x realtime mono, 0.22x stereo. `sum(map(mul, ...))` over `array` slices does the multiply-add in C. |
| The test suite leaks widgets | Same object count either way. It was `QApplication.installEventFilter` never being released — event fan-out, not memory. 4 hours → 106s. |
| Slow playback needs better clock interpolation | The FFmpeg backend reports position in coarse steps. The clock was correct and starved of input. |
| WMF's rate runs 1.8% fast, calibrate it out | A fixed ~12ms offset in the harness, divided by a short window. Real error 0.04%. |
| Stutter is the big timeline views | A 28-pixel-tall overview bar cost more than the full timeline above it. |
| Six open charts cost a third of all frames | The harness was timing the first second after load, while a Python downmix held the GIL. Steady state, six charts are +1.8ms a frame and inside budget. |

So: **do not optimise, diagnose, or "fix" a performance or timing problem
until a number says which thing to touch.** Write the harness, keep it in
`tools/`, and put the numbers in the commit message. Three of the four rows
above cost real work spent on the wrong thing.

Corollary: when a measurement surprises you, check the harness before you
believe it. The 1.8% row above was a harness artifact, and the check was
simply running it over a longer window.

### Audio: we own the transport

`audio_engine.TrackPlayer` replaced `QMediaPlayer` for song playback on
2026-08-29. Read that module's docstring before touching anything audio; the
short version is that the three things making osu!'s editor accurate all live
below the level Qt's player exposes:

- the clock is a **sample cursor** (`QAudioSink.processedUSecs()`, measured to
  be the play cursor and not the write cursor), not a signal in whole ms;
- slowing down **preserves pitch** (WSOLA in `TimeStretcher`), because osu!'s
  editor uses `AdjustableProperty.Tempo` rather than `Frequency`;
- a rate change is a **live ratio the next grain reads**, so it cannot stall.
  `QMediaPlayer.setPlaybackRate` lost 114ms of song time on a 0.25x -> 1.0x
  switch, which is what "the offset moves when I click a speed button" was.

Consequences worth remembering: there is no media backend to select any more
(`QT_MEDIA_BACKEND`, the Ogg fallback and the decoder setting are all gone, and
Qt's WMF backend is deprecated as of 6.10 anyway), the whole track is decoded
into memory (~50MB for 4:43), and the engine owns a thread that `closeEvent`
must shut down.

When porting from `ppy/osu`, port the *current* revision. This project's
`InterpolatingFramedClock` came from an older one and was missing both the
`AllowableErrorMilliseconds * Rate` scaling and the `DampContinuously` drift
recovery -- two real bugs inherited from reading a stale copy.

### The harnesses

- `tools/profile_playback.py <map> [frames] [--gimmick] [--profile]` — per-frame
  render cost against the 8.33ms budget. Reports the distribution, because
  stutter is the tail, not the mean.
- `tools/measure_audio_backend.py <backend> <audio> <rate> [seconds]` — position
  reporting granularity and rate accuracy. Use a real map's audio; FFmpeg's
  granularity turned out to be codec-dependent, so a generated WAV lied.
- `tools/measure_rate_change.py <backend> <audio> <from> <to> [seconds]` — what
  a mid-playback rate change does. Taps `QAudioBufferOutput` so "the audio went
  silent" is a number rather than a report, and reports the song time *lost* at
  the switch, which is the thing `position()` hides by looking correct again
  afterwards.
- `tools/profile_playback.py ... --gameplay --skin NAME` — the gameplay
  preview is the only view that draws the playfield, and a run without one open
  measures none of it. The full skinned playfield costs 0.11ms a frame over the
  built-in lane (1.80ms -> 1.91ms at 1882x170), which is the stretch and
  silhouette caches doing their job.
- `tools/bench_timestretch.py [rate] [seconds]` — whether the WSOLA inner loop
  keeps up. It prints the distinct splice offsets actually chosen, because a
  search that quietly degenerates looks fast for the wrong reason. **Its grain
  sizes are its own**, not `audio_engine`'s, so it measures a different amount
  of work per second of output than the real code does.
- `tools/measure_stretch_timing.py [rate ...]` — whether a transient comes out
  where the playhead says it does, which is the whole reason an editor slows
  down. The `snaps` column must be 0: a correction can place every transient
  perfectly and still arrive as a step at each grain boundary, which on screen
  is the playhead teleporting. **Its own signal is a click train and that is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jimmyreturnz/TaikoFancyEditor](https://github.com/jimmyreturnz/TaikoFancyEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
