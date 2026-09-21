---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Cos'è

Rizzo Flow: versione locale del pattern di **Jev** (TypeSafe, "System One": state non strutturato →
decisioni tipizzate con probabilità, zero generazione di testo), ispirata a **SemIf**
(`~/Git-projects/SemIf`, commit `ca3ba65`). Nessun training: si leggono i logit delle sole lettere
di risposta in un forward pass. Modello: **XHToken/Spark-X2.5-4B** (architettura `spark2_5`, pesi
originali, rev. `0bcb356…`) su **MLX** (backend Metal, CUDA o CPU; sviluppo su M4 Pro 24 GiB). Precisione
BF16 di default; Q8/Q4 quantizzati in memoria (affine, group size 64). Config d'uso normale: **Q8**.
`config.MODELS` elenca i checkpoint supportati: `4b` (default) e `1.7b` (XHToken/Spark-X2.5-1.7B, rev.
`14d6e83…`, stessa architettura/tokenizer/contesto 1M, ~3.4 GB). CLI: `--size 4b|1.7b` su
`download/decide/serve/evaluate`; `backend.load` riconosce il checkpoint da `hidden_size`
(`config.identify`) e l'ID servito diventa `rizzo-spark-x2.5-1.7b-q8`. Il 1.7B è stato provato solo su Windows/CUDA (sezione in fondo): funziona ma è molto meno accurato.

README, doc e messaggi all'utente sono in italiano; codice, commenti e docstring in inglese.
Repository pubblico: <https://github.com/Rizzo-AI-Academy/rizzo-flow> (remote `origin` via SSH, branch `main`;
il README pubblico è in inglese, quello italiano storico è in `docs/README.it.md`).

## Comandi

```bash
uv sync --extra mlx --extra test --locked        # setup Apple; --extra cuda (NVIDIA) o --extra cpu altrove
.venv/bin/rizzo devices                           # backend rilevato; su Windows gli eseguibili sono in .venv/Scripts/
.venv/bin/rizzo download                          # pesi (~8 GB) in models/Spark-X2.5-4B
.venv/bin/pytest -q                               # 41 test, ~1 s, nessun peso richiesto
.venv/bin/pytest tests/test_compat.py::test_systemone_wire_shape   # test singolo
.venv/bin/ruff check src tests scripts && .venv/bin/ruff format --check src tests scripts
.venv/bin/rizzo serve --bits 8                    # API + playground su 127.0.0.1:8017
.venv/bin/rizzo decide examples/ticket.json --bits 8
.venv/bin/rizzo evaluate benchmarks/smoke.jsonl --compare-modes --output results/local-x.json
.venv/bin/rizzo schema > request.schema.json      # rigenerare dopo modifiche a schema.py
.venv/bin/python scripts/validate_checkpoint.py --bits 8 --output results/local-validation
```

`.claude/launch.json` definisce il server di anteprima `rizzo-q8` (porta 8017). Caricare il
modello richiede ~15 s e ~5 GiB (Q8) o ~8.4 GiB (BF16): **un solo processo con pesi alla volta**,
e fermare il server prima di misurare tempi.

I test non caricano mai il checkpoint 4B: `test_service.py`/`test_compat.py` usano `FakeBackend`
+ `CharacterTokenizer` (il fake favorisce sempre il secondo candidato); `test_mlx.py` usa la vera
architettura Spark ridotta con pesi casuali. La verifica sul modello reale si fa a mano (server +
curl/playground) o con `validate_checkpoint.py`.

## Architettura (flusso di una richiesta)

`schema.py` (contratto Pydantic strict, `extra=forbid`) → `prompts.compile_request` →
`backend.SparkBackend.score` → `decisions.decode` → `responses.py` (la risposta è ri-validata
prima di uscire). `engine.Engine` orchestra tutto sotto un `Lock` (un solo modello residente:
richieste HTTP concorrenti sono serializzate; il parallelismo è *dentro* la richiesta).

- **Slot a token singolo.** Ogni candidato (opzioni + speciali `__insufficient__`,
  `__below_range__`, `__above_range__`) è una lettera maiuscola A–Z. `prompts.py` verifica che
  ogni lettera sia un token singolo e che `encode(prompt + lettera) == tokens + [id]`. Da qui il
  limite `MAX_SLOTS = 26` in `schema.py` (`require_slots`): 26 opzioni/livelli con
  `allow_abstain: false`, 25 con astensione, 24/23 ancore per `numeric`. SemIf usa lo stesso
  trucco con 16 lettere. Per andare oltre servirebbero etichette a 2 token (chain rule),
  sì/no per opzione, o etichette `AA…ZZ` (497/676 sono token singoli in Spark) — non implementato,
  decisione dell'utente: restare a 26.
- **Prefisso condiviso.** Messaggio user = `render_state(state)` + `render_question(...)`.
  Lo state è identico per tutte le domande → prefill una volta (blocchi da 512), confine del
  prefisso verificato token per token (ultimo token scartato per i merge BPE, mai dedotto dalla
  lunghezza). `branch_cache` clona le cache native (attenzione piena + sliding-window rotante),
  i suffissi vanno in microbatch (`--batch-size`, default 4, max 16) ordinati per lunghezza con
  padding a destra; si legge l'ultima posizione reale. Cache scartata a fine richiesta.
  `mode: "direct"` disattiva il riuso (riferimento di verifica).
- **Contesto.** `--ctx` (alias storico `--max-tokens`, default 8192) è il limite di token per domanda
  (state + domanda): guardia, non prenotazione di memoria. KV ≈ 36 KiB/token nel 4B (9 layer su 36 a
  attenzione piena; gli sliding sono fissi, 54 MiB), × `--batch-size` durante i microbatch perché
  `branch_cache` replica il prefisso. Lo `state` ha inoltre un tetto fisso di 256 KB in `schema.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rizzo-AI-Academy/rizzo-flow](https://github.com/Rizzo-AI-Academy/rizzo-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
