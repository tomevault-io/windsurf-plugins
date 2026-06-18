---
trigger: always_on
description: Real-time meeting intelligence system. Listens to audio, transcribes via LFM2.5-Audio, detects 4 trigger types (questions, topics, alerts, follow-ups), retrieves context via ColBERT RAG, and generates mode-aware responses using LFM2.5-1.2B-Instruct. Everything runs locally on Apple Silicon. Includes a Tauri desktop app with dual-pane UI (editable transcript + live prompts).
---

# CLAUDE.md — meeting-prompter

Real-time meeting intelligence system. Listens to audio, transcribes via LFM2.5-Audio, detects 4 trigger types (questions, topics, alerts, follow-ups), retrieves context via ColBERT RAG, and generates mode-aware responses using LFM2.5-1.2B-Instruct. Everything runs locally on Apple Silicon. Includes a Tauri desktop app with dual-pane UI (editable transcript + live prompts).

## Key Paths

```
├── coach.py                       # CLI entry point (args, startup)
├── config.yaml                    # All externalized thresholds and settings
├── lib/
│   ├── orchestrator.py            # MeetingOrchestrator — central pipeline coordinator
│   ├── config.py                  # Typed dataclass config loader
│   ├── filters.py                 # Hallucination/noise/normalization filters
│   ├── audio_capture.py           # Streaming mic/BlackHole capture (sounddevice)
│   ├── lfm2_wrapper.py            # LFM2.5-Audio subprocess wrapper (llama.cpp)
│   ├── answer_extractor.py        # Sentence extraction for grounding (fallback)
│   ├── rag_generator.py           # LFM2.5-1.2B-Instruct generation (ChatML)
│   ├── rag_engine.py              # ColBERT + Jaccard fallback orchestration
│   ├── dashboard.py               # CLI dashboard with trigger-type coloring
│   ├── triggers/                  # Multi-mode trigger engine
│   │   ├── types.py               # TriggerType enum, Trigger dataclass
│   │   ├── engine.py              # Orchestrator: runs all triggers, priority sort
│   │   ├── question_trigger.py    # Question detection (patterns + keywords)
│   │   ├── alert_trigger.py       # Watch word scanning with cooldown
│   │   ├── topic_trigger.py       # ColBERT-backed topic detection
│   │   └── followup_trigger.py    # Pause-based follow-up suggestions
│   ├── conversation/              # Conversation intelligence
│   │   ├── buffer.py              # Rolling 90s transcript + trigger routing
│   │   └── meeting_context.py     # YAML meeting context loader
│   ├── generation/                # Mode-aware generation
│   │   ├── prompts.py             # ChatML prompt templates per trigger type
│   │   ├── generator.py           # ModeAwareGenerator — trigger-routed generation
│   │   └── types.py               # GenerationResult dataclass
│   └── colbert/                   # Semantic retrieval module
│       ├── retriever.py           # LFM2-ColBERT-350M + PLAID index
│       ├── chunker.py             # Section-aware markdown chunking
│       ├── index_manager.py       # Index persistence/cache
│       └── normalizer.py          # Sigmoid score normalization
├── src/api/                       # FastAPI backend for Tauri app
│   ├── main.py                    # FastAPI server + WebSocket endpoints
│   ├── session.py                 # Session manager (bridges audio pipeline → WebSocket)
│   ├── transcript_buffer.py       # Turn-based ASR chunk accumulator
│   ├── transcript_store.py        # Append-only transcript with edit overlay + upsert
│   ├── notes_generator.py         # Structured meeting notes via LLM
│   └── routes/
│       ├── session.py             # POST /session/start|stop, GET /status, POST /reindex
│       ├── transcript.py          # WebSocket /ws/transcript (turn updates + edits)
│       ├── prompts.py             # WebSocket /ws/prompts (trigger results)
│       ├── notes.py               # Notes generate/export/save/download endpoints
│       └── context.py             # Meeting context upload
├── app/                           # Tauri + React frontend
│   ├── src-tauri/src/lib.rs       # Rust shell: spawns Python backend, manages lifecycle
│   ├── src/App.tsx                # Root component, layout, WebSocket connections
│   ├── src/components/
│   │   ├── TranscriptPane.tsx     # Left pane: turn-based editable transcript
│   │   ├── PromptsPane.tsx        # Right pane: live trigger results
│   │   ├── StatusBar.tsx          # Session controls, audio health, elapsed time
│   │   ├── MeetingSetup.tsx       # Pre-meeting context config dialog
│   │   └── NoteEditor.tsx         # Post-meeting structured notes editor
│   ├── src/hooks/
│   │   ├── useWebSocket.ts        # WebSocket connection + reconnect hook
│   │   └── useTranscript.ts       # Transcript state with turn-based upsert
│   └── src/styles/global.css      # Theme variables and animations
├── tests/                         # Colocated Python tests
│   ├── test_audio_capture.py      # Audio level detection + health diagnostics
│   ├── test_lfm2_wrapper.py       # LFM2.5-Audio output parsing
│   ├── test_session.py            # Thread-safe queue bridge + turn callbacks
│   ├── test_transcript_buffer.py  # Turn accumulation, boundaries, callbacks
│   └── test_transcript_store.py   # Append, upsert, edit overlay, export
├── models/                        # Symlink → ~/Projects/_models
├── runners/                       # llama.cpp binaries (gitignored)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chintan-projects/meeting-prompter](https://github.com/chintan-projects/meeting-prompter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
