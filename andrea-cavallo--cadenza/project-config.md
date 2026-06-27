---
trigger: always_on
description: AI-powered MIDI generator: Go 1.25, LLM-driven, progressive house / melodic techno.
---

# AGENTS.md — LLMIDI-Gen

AI-powered MIDI generator: Go 1.25, LLM-driven, progressive house / melodic techno.
**IMPORTANT:** Always read TODO.md first for the most current project status and work priorities.
This file provides architectural overview; TODO.md contains actionable tasks.

**Core Focus:** The primary goal is musical quality — generating hypnotic, varied, and musically interesting patterns, especially in offline mode (`--no-llm`). All architectural decisions serve this goal.

## Project Overview
Takes **BPM + Key** as input, produces **3 MIDI files** (bassline, arpeggio, melody). 
A shared chord progression ensures harmonic coherence. The LLM creates musical motifs; 
the renderer applies professional timing, velocity, and automation deterministically via style profiles. 
Offline mode (`--no-llm`) generates hypnotic, varied patterns algorithmically.


## Architecture

```
User (BPM + Key)
  → KeyParser
  → Step 0: Chord Progression (4 chords, shared)
  → Step 1a: MusicalPlan (style card + tension curve + motif intent)
  → Step 1: 3x parallel generators (bass, arp, melody) — all receive same chord progression + plan
  → Critic + one targeted revision round when musical scoring is weak
  → Validator (scale + range + density + chord coherence + soft musical scoring)
  → StyleProfile → Renderer
  → 3 MIDI Type-0 files
```

- **Step 0 (Chord Progression)** owns: harmonic contract — 4 chords, one per 4 bars
- **MusicalPlan** owns: producer intent — style card, tension curve, motif concept, density target, and section intent
- **LLM** owns: motif creativity, note choice, evolution arc (within chord and plan constraints)
- **Critic** owns: soft quality judgment — repetition, motif clarity, chord-tone strength, contour, density, tension arc, track separation; at most one targeted revision
- **Renderer** owns: timing offsets, velocity grids, gate lengths, CC automation, portamento
- **Validator** enforces: note range, scale membership, density, chord coherence, BPM bounds; it also reports soft musical scores
- **Offline mode** owns: seed-based algorithmic pattern generation (no API calls)

## Implementation Order

```
P0-binary
  → P0-0a (seed entropy)
  → P0-0b (key differentiation)
  → P0-0c (modal support)
  → P0-0d (LLM prompt quality)
  → P0-0e (offline hypnotic patterns)
  → P0-0f (LLM planner + critic quality)
    → P1 (README + demo — only after music sounds good)
      → P2-Phase1
      → P2-Phase2
      → P2-Phase3 + Phase4
        → P3 (coverage gaps from modal work)
```

> **Rule:** Do not start P1 (README / demo audio) until P0 music quality is solved.
> A bad demo hurts more than no demo.

## Key Directories

| Path | Purpose |
|------|---------|
| `backend/cmd/cadenza/` | CLI entry point, interactive mode |
| `backend/cmd/desktop/` | Wails desktop app entry point and AppService bindings |
| `backend/cmd/desktop/frontend/` | Vite + React + TypeScript desktop UI embedded into the Wails binary |
| `scripts/` | Local automation scripts for packaging and release builds |
| `backend/internal/theory/` | Key parsing, scales, note↔MIDI, chords, progressions |
| `backend/internal/schema/` | PatternSpec types + musical validator (with chord coherence check and soft musical scoring) |
| `backend/internal/llm/` | Provider interface, Claude (`tool_use`), Ollama (JSON schema mode), mock, retry with error classification |
| `backend/internal/renderer/` | MIDI rendering: velocity, timing, gate, sweep, evolution, portamento |
| `backend/internal/renderer/styleprofile/` | Deterministic style profiles with DynamicCurve (crescendo/arch) |
| `backend/internal/generator/` | Chord progression gen + MusicalPlan/style cards + single/multi-pattern generation + offline templates + LLM cache integration |
| `backend/internal/midi/` | MIDI Type-0 file writer with priority-based event ordering |
| `backend/internal/cache/` | SHA256-keyed disk cache (30-day TTL) |
| `backend/internal/config/` | Viper-based config loading (cadenza.yaml + env vars) |
| `backend/internal/session/` | File-based session persistence with checkpoint/eviction |
| `backend/internal/metrics/` | Prometheus counters for LLM calls, tokens, errors |
| `backend/internal/logger/` | slog setup with JSON handler for production |
| `backend/internal/models/` | Shared domain types (GenerateRequest, GenerateResult, etc.) |
| `backend/internal/prompts/` | Embedded LLM prompt templates via `//go:embed` |
| `backend/internal/service/` | Business logic layer callable from CLI and desktop |

## Conventions

- **Go 1.25** — use stdlib `log/slog` for logging, `context` for cancellation
- **Module path:** `github.com/Andrea-Cavallo/cadenza`
- **Tests:** `_test.go` next to source, table-driven, AAA pattern
- **MIDI:** Type-0, 480 ticks/beat, 120 ticks/step, CH 1 (zero-indexed 0)
- **Velocity max: 120** — never 127, it clips
- **Downbeat always on-grid** — no timing offset on step 0
- **Ghost velocity: 35-55** — never above 60
- **Portamento:** skip CC65 when tick < 10
- **Cross-platform:** `filepath.Join` for all paths, `CGO_ENABLED=0` for builds

## Go Version — Critical Rule

**`go.mod` and `.golangci.yml` must always declare the same Go version.**

```
go.mod          → go 1.25
.golangci.yml   → run: go: "1.25"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Andrea-Cavallo/cadenza](https://github.com/Andrea-Cavallo/cadenza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
