---
trigger: always_on
description: > Regole permanenti (non negoziabili, da leggere prima di toccare GUI o documentazione): GUI
---

# Mega Downloader Proxy Rotator (MDPR)

> Regole permanenti (non negoziabili, da leggere prima di toccare GUI o documentazione): GUI
> bilingue in sync — `.claude/rules/gui.md` · Doc bilingue in sync — `.claude/rules/docs-i18n.md` ·
> Doc sempre allineata al codice — `.claude/rules/documentazione-e-rilascio.md`.

## Scopo del progetto
App desktop Python+PyQt6 che scarica file da Mega.nz attraverso proxy HTTP gratuiti, con una coda di chunk a dimensione fissa (default 32 MB) scaricati da N connessioni parallele (default 10), ciascuna su un proxy diverso, e più file in parallelo (default 1, configurabile fino a 5).
Origine: test tecnico di rotazione IP (DOWNLOAD_CYCLES=3, completato e superato il 2026-05-31). Ora `DOWNLOAD_CYCLES=1`: uso normale come downloader.
Single-user, single-process, niente backend.

## Mappa moduli
```
src/
├── main.py                # entry point: QApplication + MainWindow
├── core/
│   ├── config.py          # costanti globali (timeout, soglie, paths, UA); include SPEED_SELECTION_* e VALIDATOR_SPEED_TEST_* per la selezione per velocità
│   ├── state.py           # SessionState thread-safe (pausa/annullo)
│   ├── errors.py          # UserFacingError: le eccezioni che l'utente legge portano `error_code` + `params`; `str(exc)` resta la frase ITALIANA (e' cio' che finisce nei log). `format_it()` non solleva mai (gira dentro la gestione degli errori); `error_payload()` da' codice+parametri anche per le eccezioni non nostre (ripiego `unexpected`). Le basi miste vogliono UserFacingError per PRIMO: con OSError davanti, il suo __new__ intercetta la costruzione
│   ├── error_catalog.py   # ERROR_TEXTS_IT: i 50 testi italiani degli errori, indicizzati per codice. Vive in core/ e non nei dizionari GUI perche' serve ai LOG, che restano italiani anche con l'interfaccia in inglese; `strings_it.py` li innesta come chiavi `err.*`
│   ├── telemetry.py       # telemetria "scatola nera": recorder asincrono (writer daemon) di tentativi-chunk + campioni 1Hz in logs/telemetry/<id>/; no-op se TELEMETRY_ENABLED=False
│   ├── diagnostics.py     # heartbeat periodico (INFO), RSS via psutil se disponibile / fallback GetProcessMemoryInfo (psapi) su Windows, marcatori di sessione, riga CONFIG a inizio sessione
│   ├── events.py          # EventBus opzionale (non usato dal flusso)
│   ├── logging_setup.py   # setup root logger + sys.excepthook
│   ├── failed_log.py      # logger JSONL dei link abbandonati
│   ├── download_history.py # storico JSONL download completati + extract_handle
│   ├── sources_stats.py   # logger JSONL metriche per-fonte
│   ├── version_compare.py # parse_semver/is_newer, puro stdlib (no I/O)
│   ├── branding.py        # Branding (nome/acronimo/autore/nick/link/logo): default -> cache -> remoto
│   ├── icon_loader.py     # build_app_icon(): QIcon robusta .ico->fallback .png, mai null senza log
│   ├── file_naming.py     # sanitize_folder_name() + sanitize_file_name() (nome file sicuro per Windows: caratteri riservati + device name CON/NUL/…, estensione preservata; applicata alla SORGENTE in mega_api.resolve_public_url) + final_output_dir(file_name, file_id, output_root): path finale del download (output_root = cartella scelta dall'utente, None=default); rinomina la cartella hash-based al primo resolve riuscito; + folder_job_output_dir(rel_path, output_root): cartella di destinazione ad ALBERO per i job nati da un link cartella (ri-sanifica i segmenti: è il confine col filesystem)
│   ├── disk.py            # ensure_free_space()/free_space_bytes() + InsufficientDiskSpaceError (UserFacingError+OSError, codice `disk_full`): check spazio disco PRIMA del download (errore d'ambiente: il worker abbandona subito senza bruciare i tentativi). Solo stdlib
│   ├── mega_links.py      # FONTE UNICA delle forme di URL Mega (stdlib puro, niente I/O né Crypto):
│   │                     #   file singolo, legacy, cartella `/folder/#key`, cartella con nodo selezionato
│   │                     #   (`/file/<n>`, `/folder/<n>`, legacy `#F!id!key!node`) e la forma INTERNA dei
│   │                     #   job-cartella. Ordine di match dal più specifico al più generico.
│   │                     #   `extract_handle` vive qui (download_history vi delega)
│   ├── session_store.py   # persistenza JSON dei link NON completati della sessione (save/load/clear, mai solleva) per il prompt "Riprendi sessione precedente?" all'avvio; i .part fanno il resume a livello byte
│   └── proxy_url.py       # build_proxy_url/build_proxies_dict: schema URL in base al campo protocol (http/socks4/socks5 -> socks5h) + prefisso user:pass@ opzionale se il proxy porta credenziali, solo stdlib, usato da proxy/ e downloader/; include anche cache_bust_url() (parametro query anti-cache per gli speed test) e sustained_throughput_bps() (misura pura del throughput: finestra del corpo con fallback alla finestra completa sul burst-da-buffer, evita le "bande impossibili")
├── proxy/
│   ├── sources.py         # 74 fonti pubbliche (4 html, 64 plain, 6 json/jsonl); per protocollo: 52 http, 16 socks5, 6 socks4 (campo opzionale "protocol" per fonte)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuperPHaze/mega-downloader-proxy-rotator](https://github.com/SuperPHaze/mega-downloader-proxy-rotator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
