---
trigger: always_on
description: Questo file e **vincolante**. Va rispettato al **100%, in modalita enterprise, senza eccezioni**, in ogni sessione e su ogni progetto — con il **minimo consumo di crediti** (massimo rapporto qualita/prezzo). Se una richiesta contraddice queste regole, fermati e segnalalo.
---

# CLAUDE.md — Standard operativo enterprise (universale)

Questo file e **vincolante**. Va rispettato al **100%, in modalita enterprise, senza eccezioni**, in ogni sessione e su ogni progetto — con il **minimo consumo di crediti** (massimo rapporto qualita/prezzo). Se una richiesta contraddice queste regole, fermati e segnalalo.

## 0 · Protocollo di sessione (deterministico, anti-spreco)

Esegui questi controlli **una sola volta a inizio sessione**, in ordine, poi **fermati**. Ogni passo ha una **condizione**: se gia soddisfatta, **salta senza agire** (non sprecare crediti). NON ripetere questo protocollo a ogni messaggio: si riesegue solo a nuova sessione o dopo un **evento reale** (vedi §1bis).

1. **CLAUDE.md presente?** Se il file `CLAUDE.md` non esiste nella root, o e diverso dal prompt `CLAUDE.MD` su App Control, scaricalo (`GET /rest/v1/prompts?title=eq.CLAUDE.MD&select=full_text`) e scrivilo come `CLAUDE.md` nella root. Se gia presente e identico, **non riscaricarlo**. Da qui Claude Code lo carica da solo a ogni sessione: e la tua cache, non rileggerlo dal DB ogni volta.
2. **DNA letto?** Leggi `DNA/00` e i soli file DNA pertinenti al task corrente. Non leggere tutto il DNA "per sicurezza".
3. **`.env` allineato?** Connettiti ad App Control (§1) e rigenera `.env` SOLO se manca o se mancano variabili. Se `.env` e gia completo e coerente, **non riscriverlo**.
4. **Stato progetto:** nuovo -> struttura allo standard (§4); esistente -> allinea (vale il codice). Verifica connessioni in sola lettura, senza dichiararle ok senza prova.
5. **Output:** un riepilogo breve (stato, connessioni, prossimo passo). Poi **attendi**: non avviare sviluppo, analisi o refactor senza richiesta. Prima di ogni modifica comunica gli step (§1, Regola comunicazione).

**Condizione di stop globale:** completati i 5 punti, il protocollo e CHIUSO per la sessione. Non rilanciarlo, non ri-verificare in loop. Se un passo e gia a posto, dillo in una riga e prosegui.

## 1bis · Quando ri-sincronizzare (gate eventi, non a ogni messaggio)
Ri-esegui la **riconciliazione** (§1: variabili + due link) **solo dopo un evento concreto**, non di continuo:
- hai **creato/cambiato una variabile o un segreto** nel `.env` -> caricala in App Control;
- hai fatto un **deploy** o e cambiato un URL -> aggiorna `LINK_DEPLOY` / `LINK_DEPLOY ADMIN`;
- l'utente **chiede** esplicitamente un sync.
Fuori da questi eventi, **non leggere e non scrivere** App Control: eviti azioni a vuoto e spreco di crediti. Se non c'e nulla di nuovo, non fare nulla.

## 1 · Sincronizzazione App Control (vincolante)
App Control e la **cassaforte centrale** delle variabili di ogni progetto (suo Supabase). La connessione e **remota**, indipendente dal progetto aperto.

- **Bootstrap:** file `.agent/app-control.json` nella root (in `.gitignore`), con 4 chiavi: `projectId`, `agentKey`, `appControlSupabaseUrl`, `appControlSupabaseAnonKey`. Lo leggi a inizio sessione.
- **Accesso:** Supabase REST con header `x-app-control-project-id` + `x-app-control-agent-key` + anon key. **Lettura e scrittura**, limitate al **solo** progetto della chiave.
- **Flusso:** leggi le variabili da App Control -> **generi tu il `.env`** (l'utente non scrive mai a mano nel `.env`). **Riconciliazione (ogni sync):** confronta le chiavi del `.env` reale con quelle gia in `project_env_variables` e **carica in App Control ogni variabile/segreto nuovo o cambiato** (`SESSION_SECRET`, chiavi, URL deploy/repo, qualsiasi segreto). **Escludi** solo le 7 manuali dell'utente e le derivate `VITE_*`/`SUPABASE_DB_URL`. **Colonne valore:** ogni variabile ha `value_text` (NON sensibili) e `value_ciphertext` (sensibili, `is_sensitive=true`) — MAI `value`; in LETTURA prendi il campo giusto in base a `is_sensitive` (altrimenti le sensibili escono vuote e rompi il `.env`); in SCRITTURA includi `project_id` (UUID dalla SELECT su `projects`, non lo slug, o la RLS nega) e metti il valore in `value_ciphertext`+`is_sensitive=true` se segreto, altrimenti `value_text`+`is_sensitive=false`. Se non ci sono variabili nuove, non scrivere nulla. Le nuove appaiono in App Control sotto **"Gestite da Agent"**.
- **Chi inserisce cosa:**
  - **UTENTE** (manuale, alla creazione): `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `DATABASE_URL`, `RENDER_API_KEY`.
  - **AGENT:** `GITHUB_URL` (crei tu il repo con `gh`), `GITHUB_TOKEN` e tutti i segreti generati. **Due link di deploy (dopo il deploy):** `LINK_DEPLOY` = URL pubblico/user; `LINK_DEPLOY ADMIN` = URL **reale** dell'area admin di QUESTO progetto. Il percorso/sottodominio admin **cambia da progetto a progetto**: ricavalo dal codice/config reali (route admin, deploy Render, README), **mai un suffisso fisso**. Se non esiste un'area admin separata, usa l'area di gestione reale del progetto (es. rotta protetta). Scrivi/aggiorna entrambi in `project_env_variables`; se i valori salvati non corrispondono ai link reali, correggili.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barnodemain/barnode](https://github.com/barnodemain/barnode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
