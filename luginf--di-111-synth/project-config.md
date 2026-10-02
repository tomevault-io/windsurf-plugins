---
trigger: always_on
description: A VST3/standalone emulator of the Roland D-110 multi-timbral sound module. It runs the
---

# D-110 VST Emulator - notes for Claude

A VST3/standalone emulator of the Roland D-110 multi-timbral sound module. It runs the
**real Roland firmware** (not a reimplementation of the menus/editor logic) against an
emulated LA32 sound engine (`munt`/mt32emu, D-110 fork). See `README.md` for the full
user-facing picture - architecture, ROM requirements, known limits, panel/editor
behaviour. Don't duplicate that here; this file is about how to work in the repo.

## Two CPU backends - know which one you're touching

- `plugin/Source/native/` - **`D110EmulatorNative`, the default and only actively
  developed backend.** The D-110's i8x9x/MCS-96 CPU reimplemented from scratch, zero
  MAME dependency, stepped inline on the audio thread (fixes 0-18ms MIDI jitter the
  MAME backend has). This is what CI builds (Windows/macOS) and what ships in releases.
- `plugin/Source/D110Core.*` - the original MAME-backed backend (`D110Emulator`), runs
  the firmware inside an embedded MAME `roland_d10` driver instance. Opt-in only
  (`-DD110_BUILD_MAME_BACKEND=ON`), needs a separately built MAME 0.288 tree with a
  local patch (`patches/mame_mcs96_stale_irq_level.patch`). Kept in the tree as a
  dormant fallback, not deleted, but **not to be shipped in packaging/releases
  anymore** (Alan's call, 2026-08-05) - see the memory note `feedback_native_only_releases`.
  Don't propose reviving it in `.deb`/CI unless explicitly asked.
- Both share the same JUCE plugin shell (`PluginProcessor.*`, `PluginEditor.*`) and the
  same firmware ROMs; they build as two separate plugin targets that can sit side by
  side in a DAW.

## Layout quick-reference

- `plugin/Source/PluginProcessor.*` - JUCE `AudioProcessor`: MIDI in/out routing to the
  firmware, `osMidiCollector` (`juce::MidiMessageCollector`) is the queue real MIDI
  input and UI-injected notes (`injectTestNote`) both feed, drained in `processBlock`.
  Also owns `sequencerEngine` (see below) and drives it from `processBlock`.
- `plugin/Source/PluginEditor.*` - the whole UI. `D110Panel` = the photographed panel
  with invisible hit-regions + the offscreen-rendered dot-matrix LCD. `D110EditorPane` =
  the ten-tab extended editor drawer (`Tab` enum: Parts, Tone, Rhythm, Patches,
  Timbres, Tones, System, Monitor, Soundbanks, Utility), opens downward, independently foldable.
  `D110SequencerPanel` (see below) is a third such drawer, closed by default.
  `D110MemoryCard` = the memory card slot widget.
- `plugin/Source/D110Keyboard.h/.cpp` - the on-screen test keyboard drawer (mouse piano +
  tracker-style PC keyboard input, MIDI channel/omni via right-click), independently
  foldable in the plugin, open by default. Keys light up for two independent reasons: struck
  directly here (mouse/PC keyboard - instant, no polling) or `D110KeyboardHost::isNoteActive()`
  (any note reaching the app another way - external MIDI In, sequencer playback, a DAW host
  track - a small lock-free per-note array the host's audio/MIDI thread writes to, polled by
  this component's own 30Hz timer). `midiPanic()` clears that whole array, since its CC
  64/123 "all notes off" is a controller message, not literal note-offs the array would
  otherwise ever see. `plugin/keyboard_activity_probe.cpp` covers both write paths (direct
  injection and sequencer playback) against `NonetSeqHost`. Lives outside `PluginEditor.*`
  and talks to its owner only through `plugin/Source/D110KeyboardHost.h` (note injection + its
  own persisted config), so it's also embedded, unfoldable, in `Nonet-Seq` (see below) -
  `D110AudioProcessor`
  implements that interface for the plugin, `NonetSeqHost` for Nonet Sequencer.
- `plugin/Source/sequencer/` - a D-20-style multitrack MIDI sequencer, added 2026-08-05,
  since grown to also cover step recording, undo, bar-range delete/copy/transpose, a MIDI
  Out path, and a standalone-only build of its own - see `docs/sequencer.md` for the full
  feature list, this is just the code layout. `D110SequencerEngine` is the transport/data
  model - deliberately D-110-agnostic (own internal clock, note-only
  `juce::MidiMessageSequence` per track, MIDI-file, quantize and step-recording logic),
  talking to whatever embeds it only through a `channelForTrack` callback. `D110SequencerPanel`
  is the JUCE UI drawer, talking to its host only through `D110SequencerHost` (20 methods, plus the optional `auditionTrackNote()`) -
  `D110AudioProcessor` implements that interface for the plugin, `NonetSeqHost` implements
  it for `Nonet-Seq` (**Nonet Sequencer** - CMake target and binary both `Nonet-Seq`), the
  independent sequencer app, deliberately named apart from the D-110 - Standalone-only, no
  VST3, no firmware/ROMs/plugin wrapper, just the panel/engine plus the same `D110Keyboard`
  the plugin has (for direct test-play/MIDI-routing, no fold - always visible), direct
  system MIDI In/Out, and its own settings file. 9 tracks (D-110 Parts 1-8 by their
  live SYSTEM-area channel inside the plugin, or a fixed factory-default channel map in the
  independent app, plus a rhythm track fixed on channel 10) - Nonet Sequencer only
  (`supportsExtraTracks()`) can go up to `kMaxTracks` (16): right-click above the track rows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luginf/Di-111-synth](https://github.com/luginf/Di-111-synth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
