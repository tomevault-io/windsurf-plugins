---
trigger: always_on
description: Guida per Claude Code (claude.ai/code) quando lavora in questa repo. Panoramica e struttura
---

# CLAUDE.md

Guida per Claude Code (claude.ai/code) quando lavora in questa repo. Panoramica e struttura
delle cartelle in **[README.md](README.md)**. Documenti di dettaglio:
**[docs/TASSONOMIA_TAG.md](docs/TASSONOMIA_TAG.md)** (i 22 tag) e
**[docs/DATASET.md](docs/DATASET.md)** (composizione completa di train/validation).

## Cos'è questo progetto

Pipeline per addestrare un modello **mmBERT** (`jhu-clsp/mmBERT-base`) a fare **token
classification di PII**, con focus su **testi legali italiani** (atti, contratti, sentenze)
ma con training **multilingue**. Obiettivo finale: anonimizzare documenti **in locale**
prima di mandarli a LLM closed (anonymize → placeholder + dizionario reversibile locale →
API → ricostruzione), per studi legali / compliance GDPR.

Scelta di mmBERT (encoder multilingue, architettura ModernBERT, context nativa 8192) e non
ModernBERT vanilla perché quest'ultimo è quasi solo inglese.

## Ambiente — vincoli critici e non ovvi

GPU **RTX 5060 Ti (Blackwell, sm_120)** su Windows; Python in `D:\programmi\python`.
Questi punti fanno fallire tutto se ignorati:

- **torch DEVE essere build cu128**: `torch 2.11.0+cu128` (da `https://download.pytorch.org/whl/cu128`).
  Le build cpu/cu121 non supportano sm_120 → `torch.cuda.is_available()` False o crash.
- **torchvision/torchaudio vanno disinstallati** se a versioni vecchie: rompono l'import di
  transformers con `operator torchvision::nms does not exist`. Non servono qui.
- **accelerate ≥ 1.14** (transformers 4.57 chiama `unwrap_model(keep_torch_compile=...)`).
- Windows: nel Trainer **`dataloader_num_workers=0`** (altrimenti `RuntimeError ... bootstrapping`).
- **seqeval non si compila** (bug setuptools_scm) → metriche entity-level (P/R/F1) calcolate
  a mano dentro gli script, nessuna dipendenza.
- Dipendenze extra installate: **`wandb`** (tracking) e **`python-dotenv`** (legge `.env`).
- **Log da PowerShell**: redirezioni `>` e `Tee-Object` scrivono **UTF-16** → leggerli con
  `Get-Content`/`python`, NON con lo strumento Read. Gli script forzano `sys.stdout` UTF-8.

## Mappa della repo

Struttura completa in [README.md](README.md). In breve: codice in `src/`, dati in `dataset/`,
modelli in `models/<versione>/`, artefatti dei run in `experiments/<run>/`, doc in `docs/`.
**I path negli script sono assoluti (risolti da `__file__`): girano da qualsiasi CWD.**

**Pipeline dati — `src/data_pipeline/` (ordine di esecuzione):**
- `llm_template_bank.py` — Gemini scrive documenti legali con soli segnaposto `{SLOT}` →
  `dataset/synthetic/legal_templates.json` (72 template). Guard scarta i template con nomi inline.
- `generate_synthetic_pii.py` — inietta nei template dati con **checksum validi** (CF/PIVA/IBAN)
  → `dataset/synthetic/synthetic_pii_it_200k.jsonl` (`tokens` + `bio_labels`).
- `augment_real_pii.py` — inietta entità sintetiche in **frasi reali** Ai4Privacy (it) in posizioni
  variabili → `dataset/synthetic/synthetic_pii_it_realaug.jsonl`. Spezza il legame template/posizione.
- `prepare_deepmount.py` — rimappa `DeepMount00/pii-masking-ita` (56 tipi Faker IT) sui 22 tag →
  `dataset/processed/deepmount_pii_it_{train,test}.jsonl`.
- `build_validation.py` — **unica validation reale** → `dataset/validation/validation_real.jsonl`.
- `build_subset.py` — subset stratificati (multilingua + tag) per smoke test →
  `dataset/subsets/{train_subset_10k,val_subset_5k}.jsonl`.

**Training — `src/training/`:**
- `train_pii.py` — fonde tutte le fonti, addestra, salva il modello, plotta la loss, stampa
  metriche train/val, logga su **W&B** (run `rizzo-pii:0.3B-v{VERSION}`). **Modello versionato**:
  ogni run grande salva in `models/rizzo-pii-0.3B-v{VERSION}/` (storico, niente sovrascrittura;
  `MODEL_VERSION` in cima al file o `--version`; storia in `models/registry.json`). Vedi "Parametri".
- `test_pii.py` — inferenza CLI sul modello salvato (entità + testo anonimizzato). Risolve in automatico
  l'**ultima** versione `rizzo-pii-0.3B-v*` (fallback al vecchio non versionato → legacy; override `PII_MODEL_DIR`).

**Utility / ispezione (read-only) — `src/inspect/`:**
- `validate_checksums.py` — ricalcola i checksum CF/IBAN/PIVA; **blueprint della rete regex+checksum**
  da affiancare al modello in produzione.
- `inspect_ai4privacy.py` (conteggi lingue/tag), `inspect_lengths.py` (lunghezze), `inspect_no_iban.py`.

**App di anonimizzazione locale — `src/app/` (+ packaging in `docs/BUILD.md`):**
- `app.py` — server Flask + **UI**: testo o PDF, chunking con overlap, offset globali + dedup.
  Anonimizzazione **reversibile** (ogni PII → `[FULLNAME_1]`/`[IBAN_1]`… + dizionario locale; tab
  "Ripristina"). Affianca al modello una **rete regex/checksum** (EMAIL/TELEFONO/IBAN/CF/PIVA/carta/
  importo/targa; IBAN/CF/PIVA/carta validati con checksum, che ha priorità sul modello). `APP_VERSION`.
- `serve.py` — entry **headless** (solo Flask, niente browser): è il backend dell'app Tauri; log su
  `%LOCALAPPDATA%\rizzo-pii\backend.log`. `desktop_app.py` — entry PyInstaller legacy (apre il browser).
  `assets/` — mascotte (il riccio) + icone. `smoke_app.py`, `make_test_pdf.py`.

**App desktop Tauri — `tauri/`:** finestra nativa **Rizzo PII** (WebView2) che lancia il backend

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rizzo-AI-Academy/rizzo-pii](https://github.com/Rizzo-AI-Academy/rizzo-pii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
