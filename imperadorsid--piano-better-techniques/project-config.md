---
trigger: always_on
description: A Rails 8.1 app that helps beginners learn piano through AI-generated song guides, MIDI keyboard practice, and progress tracking. The AI layer calls the Claude API (`claude-sonnet-4-6`) to produce structured study content (overview, song map, hand positioning, difficult sections, harmony) for each song.
---

# CLAUDE.md — Piano Learner

## Project Overview

A Rails 8.1 app that helps beginners learn piano through AI-generated song guides, MIDI keyboard practice, and progress tracking. The AI layer calls the Claude API (`claude-sonnet-4-6`) to produce structured study content (overview, song map, hand positioning, difficult sections, harmony) for each song.

**Stack:** Ruby on Rails 8.1 · SQLite · Solid Queue / Cable / Cache · Hotwire (Turbo + Stimulus) · Importmap · VexFlow 5.0 · Chart.js 4 · Vitest · RSpec · Claude API (Anthropic)

---

## Features

### Song Library (`/songs`)
Grid of all songs with `import_status: "ready"`. Each card shows title, composer, difficulty badge (1–5), BPM, key signature, time signature, and links to Practice and Analysis.

### Song Detail (`/songs/:id`)
Header with title/composer, info cards (BPM, Key, Time Signature), detected chord progression, and a list of practice parts (name, hand, note count) each with a "Start Practice" button.

### AI Song Analysis (`/songs/:id/analyze`)
Five AI-generated sections (overview, song map, hand positions, difficult sections, harmony) plus structured data panels (chord grid, difficulty sections, dynamics map, hand separation). Turbo Stream updates in real time when AI generation completes. Regenerate button re-triggers the job.

### Practice Mode (`/song_parts/:id/practice_sessions/:id`)
BPM-based auto-advance practice with:

- **Count-in**: 1-measure countdown ("1, 2, 3, 4") at song BPM before music starts
- **Staff notation**: VexFlow renders 3 measures at a time with treble clef and bar lines. A red vertical playhead line sweeps across at BPM speed. Staff auto-scrolls to the next page of 3 measures when the playhead crosses the boundary. Notes not yet reached render dark (#333); correct notes turn green (#4CAF50); missed or incorrect notes turn red (#f44336)
- **Virtual keyboard**: 61 keys (C2–C7) with octave labels. Flashes green on correct play, red on wrong note
- **MIDI input**: WebMIDI bridge captures note-on/off events from connected MIDI keyboards
- **Timing engine**: `requestAnimationFrame` loop tracks `performance.now()` elapsed time. Each note has a beat window `[beat, beat + dur)`. Playing the correct MIDI note inside the window scores "correct"; wrong MIDI scores "incorrect"; window passing with no input scores "missed"
- **Restart**: Resets counters, clears staff colors/playhead, removes Session Complete card, restarts count-in
- **Completion**: Auto-completes when playhead passes the last note. Sends `correct_notes`, `incorrect_notes`, `notes_reached` to server. Server runs `SessionScorer` and renders a Turbo Stream completion card with composite score breakdown

### Scoring System
Each completed session is scored across three dimensions:

| Dimension | Weight | Calculation |
|---|---|---|
| **Accuracy** | 50% | `correct / (correct + incorrect) × 100` |
| **Timing** | 25% | Consistency of response times (median absolute deviation from median) |
| **Dynamics** | 25% | Average per-note velocity deviation from expected (0–127 scale) |

Composite score = weighted sum. If velocity data is unavailable, its 25% is redistributed equally to accuracy and timing. Timing/velocity scores only use correct note attempts.

### Practice History (`/practice_sessions`)
Table of all sessions showing song, part, date, notes reached, accuracy %, composite score, and completion status. Color-coded: green ≥80%, yellow ≥50%, red <50%.

### Dashboard (`/dashboard`)
Summary metrics (total sessions, average accuracy, average composite, unique songs), Chart.js line graph of accuracy over time, and 5 most recent sessions table.

### MIDI Setup (`/midi_setup`)
Live device connection status, device list, last note played display with velocity bar. Troubleshooting tips.

---

## Key Architecture

### Models & Associations

```
Song
  ├─ has_one  :song_analysis      (AI-generated content + structured data)
  ├─ has_many :song_parts         (left/right/both hand note data)
  └─ has_many :practice_sessions
       └─ has_many :session_attempts
```

### Note Data Structure

Each note in `song_parts.notes_data` (JSON array):
```json
{ "pos": 0, "midi": 60, "name": "C4", "dur": 1.0, "vel": 80, "beat": 0.0 }
```
- `pos`: sequential index (0-based)
- `midi`: MIDI note number (0–127, middle C = 60)
- `name`: note name with octave (e.g., "C#4")
- `dur`: duration in beats (1.0 = quarter, 2.0 = half, 0.5 = eighth)
- `vel`: velocity (0–127)
- `beat`: absolute beat position in song timeline

### Song Fields
`title`, `composer`, `tempo_bpm` (integer), `time_signature` (string, e.g. "4/4"), `key_signature`, `difficulty` (1–5), `import_status` ("ready" shown in library)

### Stimulus Controllers

| Controller | Responsibility |
|---|---|
| `practice_controller` | BPM timing engine, count-in, auto-advance, note evaluation, scoring, session completion |
| `staff_controller` | VexFlow staff rendering (3 measures), playhead animation, note result coloring |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ImperadorSid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
