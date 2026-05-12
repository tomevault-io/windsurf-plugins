---
trigger: always_on
description: This is a terminal-based backing track player written in Go that uses YAML-based DSL (Domain-Specific Language) called BTML (Backing Track Markup Language) to define complete backing tracks for guitar practice.
---

# Claude Project Context: Backing Tracks

## Project Overview

This is a terminal-based backing track player written in Go that uses YAML-based DSL (Domain-Specific Language) called BTML (Backing Track Markup Language) to define complete backing tracks for guitar practice.

**Current Version:** v0.7

**Purpose:** Enable guitarists to create and play full-band backing tracks (chords, bass, drums) from simple YAML files, with real-time visual display showing current chord and beat.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│ BTML File (.yaml)                                       │
│ - Track metadata (tempo, key, time signature)          │
│ - Chord progression (pattern string)                   │
│ - Bass configuration (style, swing)                    │
│ - Drums configuration (presets or Euclidean rhythms)   │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Parser (parser/parser.go)                               │
│ - Parses YAML into Go structs                          │
│ - Validates and sets defaults                          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Display (display/terminal.go)                           │
│ - Shows track info, chord grid, bass/drum info         │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ MIDI Generation (midi/)                                 │
│ ├─ generator.go: Main MIDI file creation               │
│ ├─ bass.go: Bass pattern generation                    │
│ └─ drums.go: Drum pattern generation (w/ Euclidean)   │
│                                                         │
│ Output: 3-track MIDI file                              │
│ - Track 0: Tempo metadata                              │
│ - Track 1: Chords (channel 0, piano)                   │
│ - Track 2: Bass (channel 1, fingered bass)             │
│ - Track 3: Drums (channel 9, GM drum map)              │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Live Display (display/live.go)                          │
│ - Real-time chord and beat visualization               │
│ - Updates 10x/second via goroutine                     │
│ - Visual metronome, progress bar                       │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Player (player/fluidsynth.go)                           │
│ - Executes FluidSynth via exec.Command                 │
│ - Manages live display lifecycle                       │
│ - Synthesizes MIDI to audio using SoundFont            │
└─────────────────────────────────────────────────────────┘
```

## Key Components

### 1. Parser (`parser/parser.go`)

**Responsibility:** Parse BTML YAML files into Go structs

**Key Types:**
- `Track`: Root structure containing all track data
- `TrackInfo`: Metadata (title, key, tempo, time signature, style)
- `ChordProgression`: Pattern string, bars per chord, repeat count
- `Bass`: Style (root, root_fifth, walking, swing_walking), swing ratio
- `Drums`: Style presets OR explicit patterns
- `DrumPattern`: Can be Euclidean rhythm OR explicit beats
- `EuclideanRhythm`: Algorithmic rhythm (hits, steps, rotation)

**Important Methods:**
- `LoadTrack(filename)`: Main entry point, loads and validates BTML
- `GetChords()`: Expands pattern string into chord sequence with repeats
- `TotalBars()`: Calculates total bars in progression

### 2. MIDI Generation (`midi/`)

#### `generator.go`
**Responsibility:** Create multi-track MIDI file

**Key Functions:**
- `GenerateFromTrack(track)`: Main orchestrator
- `getChordVoicing(symbol)`: Converts chord symbols to MIDI notes
- `parseRoot(symbol)`: Extracts root note (C, D, E, etc.)
- `parseQuality(symbol)`: Extracts chord quality (7, maj7, m7, etc.)

**MIDI Structure:**
- Uses `gitlab.com/gomidi/midi/v2/smf` for MIDI file generation
- 480 ticks per quarter note
- 1920 ticks per bar (4/4 time)
- Channels: 0=chords, 1=bass, 9=drums (GM standard)

#### `bass.go`
**Responsibility:** Generate bass note patterns

**Styles:**
- `root`: Simple root notes on downbeats
- `root_fifth`: Root on beat 1, fifth on beat 3
- `walking`: Root, 3rd, 5th, 7th pattern
- `swing_walking`: Walking bass with swing feel (configurable ratio)

**Swing Implementation:**
- `swing=0.5`: Straight feel (50/50)
- `swing=0.6`: Slight swing (60/40)
- `swing=0.67`: Triplet swing (67/33)

**Key Functions:**
- `GenerateBassLine()`: Main generator
- `getThird()`: Returns major/minor 3rd based on chord quality
- `getSeventh()`: Returns appropriate 7th interval

#### `drums.go`
**Responsibility:** Generate drum patterns

**Preset Styles:**
- `rock_beat`: Kick 1,3 | Snare 2,4 | 8th note hihat
- `shuffle`: Blues shuffle with triplet feel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ako/backing-tracks](https://github.com/ako/backing-tracks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
