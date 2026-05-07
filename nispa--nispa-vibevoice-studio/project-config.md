---
trigger: always_on
description: > Contesto di progetto per Claude Code. Aggiornato: 2026-03-25.
---

# CLAUDE.md — Nispa VibeVoice Studio

> Contesto di progetto per Claude Code. Aggiornato: 2026-03-25.

## Cos'è il progetto

Nispa VibeVoice Studio — app desktop offline per:
- **Voiceover AI** da sottotitoli (.srt/.vtt) o script liberi
- **Traduzione AI** offline (NLLB-200 + Ollama)
- **Voice cloning** con Qwen3-TTS e VibeVoice

**Stack:** React 19 + Vite + TypeScript + Tailwind v4 (frontend) | FastAPI + SQLite + PyTorch (backend)

**Avvio:** `start.bat` (lancia backend uvicorn + frontend npm dev + browser)

**Versione corrente:** 0.7.0

---

## Ambiente Hardware

- GPU: **NVIDIA RTX 5070 Ti Laptop** (Blackwell, sm_120, 16GB VRAM)
- CUDA: **13.2**
- PyTorch: **2.10.0+cu130** (installato da `https://download.pytorch.org/whl/cu130`)
- Flash Attention: installato con `pip install flash-attn --no-build-isolation`
- **IMPORTANTE:** Non usare `cu124` o build stabili standard — non supportano sm_120

---

## Struttura del Progetto

```
nispa-voiceover/
├── backend/
│   ├── main.py                    # FastAPI app, startup, serve audio files
│   ├── api/routers/
│   │   ├── tasks.py               # SSE generation tasks, VRAM batching, /tasks/active
│   │   ├── system.py              # /health, /status, VRAM info, maintenance
│   │   ├── voices.py              # gestione voci
│   │   ├── jobs.py                # archivio job (DB)
│   │   ├── generation.py          # generate-segment singolo
│   │   └── translation.py         # traduzione
│   ├── core/
│   │   ├── tts_provider.py        # MultiModelProvider (lazy init, orchestrator)
│   │   ├── tts/
│   │   │   ├── vibe_provider.py   # VibeVoice — batching nativo HuggingFace
│   │   │   └── qwen_provider.py   # Qwen3-TTS — lru_cache ref audio, language detection
│   │   ├── queue_manager.py       # TTSQueueManager, task eviction 10min, get_active_task()
│   │   ├── config.py              # config_manager → data/settings.json
│   │   ├── audio_storage.py       # salvataggio segmenti WAV su disco
│   │   └── parser.py              # parse SRT/VTT/script
│   ├── db/
│   │   ├── database.py            # SQLite init, get_job, update_job
│   │   └── models.py              # Pydantic models (extra="ignore" per compat legacy)
│   └── requirements.txt           # torch==2.10.0 cu130
├── frontend/src/
│   ├── features/subtitle/
│   │   ├── context/SubtitleContext.tsx      # state centrale subtitle feature
│   │   ├── components/GenerationControls.tsx # SSE, session recovery, persistActiveTask
│   │   ├── components/GenerationProgressDisplay.tsx
│   │   └── hooks/useGenerationProgress.ts   # ETA calc (Xs / Xm Xs / Xh Xm)
│   ├── features/script/           # script voiceover feature
│   ├── context/GlobalContext.tsx  # voices, models, isProcessing globale
│   ├── services/ttsApi.ts         # API calls (getActiveTask, submitGenerationTask, ecc.)
│   └── utils/audio.ts             # base64ToBlobUrl, filePathToHttpUrl, serializeAudioUrl
├── data/
│   ├── settings.json              # config persistita (batch_overrides, ecc.)
│   ├── voices/                    # file WAV voci + .txt trascrizioni
│   ├── audio-rendering/           # segmenti WAV generati, per job
│   └── outputs/                   # file audio finali esportati
├── start.bat                      # launcher (include SoX PATH patch via delayed expansion)
└── install.bat                    # installer v0.7.0 (torch cu130 + flash-attn)
```

---

## Decisioni Architetturali Chiave

### TTS Engine
- **Lazy loading**: i modelli non vengono caricati all'avvio — solo al primo `synthesize()` call
- **VibeVoice**: batching nativo con singolo `processor()` + `model.generate()` sulla batch intera
- **Qwen3-TTS**: `lru_cache` su `_get_voice_ref()`, language detection per-segmento, `soundfile.write()` invece di `torchaudio.save()` (evita dipendenza torchcodec)
- **MultiModelProvider** (`tts_provider.py`): orchestrator, seleziona provider da model_name

### VRAM & Batching
- Budget: `free_vram * 0.60` (40% headroom per KV cache/attention peaks)
- Config per modello in `_MODEL_VRAM_CONFIG`: `(cost_gb, peak_multiplier, max_batch)`
- OOM recovery: dimezza batch, raddoppia stima costo, retry sequenziale
- First-batch profiling: misura costo reale VRAM e aggiorna stima
- Log VRAM: stampa solo quando il batch size cambia (no spam)
- User override: `settings.json` → `tts.batch_overrides[model_name]`

### Audio Storage
- Segmenti salvati come WAV in `data/audio-rendering/{slug}_{id}/`
- `audioUrl` nel DB = path relativo su disco
- Serviti via `GET /audio-files/{job_folder}/{filename}` con `Cache-Control: no-store`
- SSE non invia più base64 — invia `audio_url` HTTP

### Session Recovery (refresh durante generazione)
- `task_id` + `job_id` salvati in `sessionStorage` all'avvio generazione
- Al mount di `GenerationControls`, chiama `GET /api/tasks/active`
- Se il task è ancora attivo, riconnette SSE automaticamente
- `sessionStorage` pulito a completamento / errore / cancel

### Task Lifecycle
- Task in memoria nel `queue_manager.tasks` dict
- Eviction automatica 10 min dopo completamento/fallimento/cancel
- `GET /api/tasks/active` → restituisce il task QUEUED/PROCESSING corrente

### CORS & Cache
- `Cache-Control: no-store` su tutti gli endpoint audio (fix Edge cache poisoning)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nispa/nispa-vibevoice-studio](https://github.com/nispa/nispa-vibevoice-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
