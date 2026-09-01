---
trigger: always_on
description: K-Hub è una piattaforma web per aggregare eventi di rental karting in Italia e (in roadmap) tracciare le statistiche dei piloti a livello nazionale. Target: piloti rental, in particolare neofiti.
---

# K-Hub — Contesto Progetto

## Cos'è
K-Hub è una piattaforma web per aggregare eventi di rental karting in Italia e (in roadmap) tracciare le statistiche dei piloti a livello nazionale. Target: piloti rental, in particolare neofiti.

## Stack reale (NON Flutter — attenzione, vecchi prompt lo presumevano)
- **Frontend**: React 19 + Vite + Tailwind CSS v4, react-router-dom v7, lucide-react per le icone. Stile "brutalist/sportivo" con CSS custom in `frontend/src/index.css` (variabili tipo `--castrol-red`, classi `card-snappy`, `btn-snappy`).
- **Backend**: Supabase (PostgreSQL + Auth), RLS attiva. Schema base in `database_schema.sql`, esteso dalle migration in `migrations/` (001→004, applicate al DB live a mano via SQL editor: il CLI Supabase non ha access token su questa macchina — mai tentare scritture di test sul DB di produzione).
- **Scraper**: Python, 5 fonti orchestrate da `run_all.py`: SWS (Playwright + playwright-stealth + BeautifulSoup, Chrome reale non-headless, `scraper_base.py`), WeRace/XRace/KRM (HTTP/JSON o BeautifulSoup), RKC ASI (`rkc_asi_scraper.py`, API JSON pubblica del sito — plugin "The Events Calendar", no Playwright necessario). Scrive nella tabella `events` con upsert dedup su `(source_url, event_date)` e arricchisce region/format a scrape-time. Usa la libreria interna `it-scraper-toolkit` (repo locale `C:\Users\FCD\Documents\it-scraper-toolkit`, GitHub privato `Fuchs665/it-scraper-toolkit`, installata editable via requirements.txt): `configure_stdio()` in `scraper_base` rende le print encoding-safe su Windows (niente più crash da emoji/accenti), `HttpClient` con retry/backoff + rate limit 1 req/s + user-agent onesto "K-Hub-Scraper/0.1" per le 4 fonti HTTP (verificato: nessun sito lo blocca), `parse_italian_date` per le date SWS/KRM. `run_all.py --dry-run` esegue tutto senza scrivere sul DB e segnala i duplicati cross-fonte (stessa data+pista da fonti diverse, report-only).
- **Deploy/vincoli**: solo free tier Supabase e Vercel. Nessuna spesa.

## Stato attuale (luglio 2026 — Step 2, 3, 4, 5, 6 e 7 completati)
- DB live (ref `yiysqhbtmjdpooznsgbg`) allineato a `migrations/001→006`: `tracks` (+ seed 14 piste/6 regioni), `events` (+ region, format, created_by, series), `profiles`, `race_results`, `lap_times`, `regulations`; vista `v_pilot_leaderboard` (non più usata in UI, vedi Step 4); RPC `get_pilot_stats(p_pilot_id)` e `get_event_standings(p_event_id)`; indice UNIQUE completo su `events(source_url, event_date)`.
- Frontend: TUTTE le query dati passano dal layer repository in `frontend/src/lib/` (`eventsRepository`, `pilotsRepository`, `tracksRepository`, `resultsRepository`, cache TTL in `cache.js`); il client Supabase esiste solo in `lib/supabase.js` e le pagine lo usano direttamente solo per l'auth.
- Pagine React: Home, Calendar (chip Tipologia Gara + Filtri Avanzati collassabile, chip filtri attivi rimovibili, vista Lista raggruppata per bucket data e vista Mese con griglia cliccabile), TracksDirectory (mappa SVG Italia cliccabile), RkcAsi (tab per regione con dati reali `events.series='rkc_asi'`, vedi Step 7), GuidaRental (guida neofiti, sostituisce la vecchia Leaderboard nazionale), Dashboard pilota (KPI + trend punti + storico), EventDetails (classifica + accordion tempi), OrganizerDashboard (inserimento eventi + risultati/tempi con import bulk), Auth.

## Problemi noti / debito tecnico
1. **RLS lasca su risultati**: le policy su `race_results`/`lap_times` sono `FOR ALL TO authenticated WITH CHECK (true)` — qualsiasi utente loggato può inserire/modificare risultati altrui. Accettato per l'MVP; in futuro serve un ruolo organizer.
2. **Scraper non testato post-refactor**: l'upsert dedup e l'arricchimento region/format non hanno ancora avuto un run reale contro il DB live. Nota 2026-07-20: la pipeline di estrazione è stata verificata con `run_all.py --dry-run` (37 eventi da 4 fonti, zero crash) dopo l'integrazione di it-scraper-toolkit, ma il ramo di INSERT resta non testato; inoltre i file `.env.local` (scraper e frontend) non sono presenti su disco, quindi un run reale richiede prima di ripristinarli.
3. **Nessuna UI di modifica/cancellazione risultati**: OrganizerDashboard permette solo l'inserimento; errori di battitura si correggono solo dal DB.
4. **Artefatti di debug tracciati in git**: `scraper/sws_races.html` + `sws_races_files/`, `debug_screenshot.png` — da pulire dopo aver verificato che l'HTML non serva come fixture.
5. **Possibile duplicazione cross-fonte per le tappe RKC ASI**: alcune tappe condividono la pista con eventi già scrapeati da altre fonti (es. Misanino KCE, già noto a KRM) ma con `source_url` diverso — la dedup upsert è su `(source_url, event_date)`, quindi non rileva duplicati tra fonti diverse per lo stesso giorno/pista. `run_all.py` ora li segnala automaticamente a ogni run (report-only via `toolkit.dedupe.find_duplicates` su data+pista, filtrato su domini sorgente diversi); al run del 2026-07-20 nessun duplicato cross-fonte in pratica. Se in futuro il report ne trova, decidere quale fonte vince prima di automatizzare la rimozione.

## Convenzioni
- Lavorare in iterazioni: proporre → conferma → implementare.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fuchs665/K-Hub](https://github.com/Fuchs665/K-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
