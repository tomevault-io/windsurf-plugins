---
trigger: always_on
description: App desktop locale (Windows + macOS): qualsiasi video parlato → trascrizione fedele → traduzione editoriale opzionale → SRT per DaVinci Resolve.
---

# Subber

App desktop locale (Windows + macOS): qualsiasi video parlato → trascrizione fedele → traduzione editoriale opzionale → SRT per DaVinci Resolve.

Default UI: lingua parlata **Rileva automaticamente**, sottotitoli **Italiano**. Accetta qualsiasi lingua Whisper. I file usano il codice lingua reale: `{nome}.{lang}.txt`, `{nome}.{lang}.srt`.

## Pipeline (ordine fisso)

`video → FFmpeg audio → faster-whisper (task=transcribe) → traduzione contestuale su blocchi (prev + curr + next) se le lingue differiscono → formatter SRT → export`

Mai `task=translate` di Whisper come unico output. Mai tradurre l’audio diretto in italiano.

Output per video: cartella `{outputDir}/{stem}/` con `{nome}.{lingua}.txt`, `{nome}.{lingua}.srt`, `{nome}.{output}.srt`, `{nome}.json`. In modalità video anche `{stem}.{lang}.{res}.{ext}`.

## Stack

Tauri 2 + React + TypeScript + Rust. Worker Python (faster-whisper) dal task 3. Architettura minima.

## Task

1. **Fatto:** drag & drop video + gestione file + UI di base  
2. **Fatto:** estrazione audio FFmpeg (WAV 16 kHz mono)  
3. **Fatto:** trascrizione + timestamp (faster-whisper, `task=transcribe`, qualsiasi lingua)  
4. **Fatto:** export `{lang}.txt` / `{lang}.srt`  
5. **Fatto:** traduzione contestuale (default parlato auto, sottotitoli IT, lingue selezionabili)  
6. **Fatto:** Formatter SRT nella lingua di output  
7. **Fatto:** Export `{output}.srt` + `.json` in cartella per video  
8. **Fatto:** Glossario (ASR + traduzione)  
9. **Fatto:** Progress / errori  
11. **Fatto:** Modalità prodotto SRT | video in Home  
12. **Fatto:** Overlay didascalie + griglia magnetica  
13. **Fatto:** Stile didascalie (font, colori, posizione)  
14. **Fatto:** Export video sottotitolato (FFmpeg, fino a 4K)

## Contratti invoke

- `inspect_videos(paths: string[])` → `{ videos: VideoFile[], skipped: { path, reason }[] }`
  - `VideoFile`: `path`, `name`, `sizeBytes`, `parentDir`
  - Accetta file video o una cartella (solo primo livello)
  - Estensioni: `mp4 mov mkv m4v avi webm mpg mpeg wmv`
- `extract_audio(videoPaths: string[], outputDir: string)` → `{ ffmpegPath, items[] }`
  - `items`: `videoPath`, `audioPath`, `durationSecs`, `error`
  - Evento `extract-progress`: `videoPath`, `audioPath`, `status` (`extracting` | `done` | `error`), `message`, `percent`
  - Output: `{outputDir}/{stem}.wav` — PCM 16-bit, 16 kHz, mono
  - Se il WAV esiste già ed è più recente del video, l’estrazione viene saltata
- `transcribe_audio(items, language, quality, glossary)` → `{ items[] }`
  - `items` in: `{ videoPath, audioPath }`
  - `items` out: `{ videoPath, jsonPath, segmentCount, error }`
  - Evento `transcribe-progress`: `videoPath`, `status` (`transcribing` | `done` | `error`), `message`, `percent`
  - Output: `{outputDir}/{stem}.asr.json` (segmenti con start/end/text/confidence)
  - Qualità: `fast` → base, `balanced` → small, `max` → large-v3
  - Worker: `worker/transcribe.py` (mai `task=translate`; `auto` = rilevamento lingua)
- `export_source(items)` → `{ items[] }`
  - `items` in: `{ videoPath, jsonPath }`
  - `items` out: `{ videoPath, folderPath, txtPath, srtPath, language, error }`
  - File in `{outputDir}/{stem}/`: `{stem}.{lang}.txt` (trascrizione intera) e `{stem}.{lang}.srt` (max 2 righe, ~42 caratteri)
  - Formatter: `worker/subtitles.py` (riusato per la lingua di output)
- `translate_segments(items, targetLanguage, glossary)` → `{ items[] }`
  - `items` in: `{ videoPath, jsonPath, sourceLanguage? }`
  - `items` out: `{ videoPath, trlPath, sourceLanguage, targetLanguage, segmentCount, error }`
  - Evento `translate-progress`: `videoPath`, `status` (`translating` | `done` | `error`), `message`, `percent`
  - Output: `{outputDir}/{stem}.trl.json` (stessi timestamp; `text` originale, `translated`)
  - Contesto: segmento precedente + corrente + successivo
  - Worker: `worker/translate.py` — NLLB-200 locale, mai Whisper `task=translate`
  - Lingua sorgente dal file `.asr.json` (rilevata o scelta). Se uguale all’output, copia senza tradurre
  - Glossario: placeholder in traduzione (GLOSS00…) e forma canonica ripristinata; in ASR `initial_prompt` + correzione maiuscole/minuscole
- `save_script(items)` → `{ items[] }`
  - `items` in: `{ videoPath, path, segments }`
  - Aggiorna `.asr.json` / `.trl.json` e rigenera la cartella di export
- `engine_status(quality?)` → `{ ffmpegOk, ffmpegPath, pythonOk, pythonPath, whisperOk, translateOk, whisperReady, translateReady, modelsReady, whisperModel }`
  - `whisperOk` / `translateOk`: pacchetti Python
  - `whisperReady` / `translateReady` / `modelsReady`: file modello in cache HuggingFace
- `prepare_models(quality, parts?)` → `{ whisperReady, translateReady, modelsReady, whisperModel }`
  - `parts`: `all` | `whisper` | `translate` (default `all`)
  - Evento `prepare-progress`: `status`, `part`, `message`, `percent`
  - Worker: `worker/prepare.py` — scarica Whisper (qualità scelta) e NLLB prima del lavoro
  - All’avvio l’app installa Python (uv + venv in app data) e i pacchetti, poi scarica Whisper e NLLB. Loader a schermo intero finché `modelsReady`.
- `export_output(items)` → `{ items[] }`
  - `items` in: `{ videoPath, trlPath }`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreaZero/subber](https://github.com/AndreaZero/subber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
