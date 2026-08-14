---
trigger: always_on
description: Promemoria per lavorare bene su questo progetto. Leggi prima di iniziare un task.
---

# CLAUDE.md — istruzioni per l'assistente

Promemoria per lavorare bene su questo progetto. Leggi prima di iniziare un task.

## 🔚 Comando «chiudo» (rituale di fine sessione) — ISTRUZIONE PERMANENTE

Quando l'utente scrive **`chiudo`** (da solo o in una frase), PRIMA di rispondere:

1. **Analizza la chat corrente** ed estrai ciò che sarà utile al *me* futuro per **decidere meglio e più in fretta**:
   - decisioni prese **+ il perché**; opzioni **scartate** e il motivo (per non riproporle);
   - **preferenze e stile di lavoro** dell'utente emersi;
   - **fatti verificati** (con data, perché deperibili) e link/fonti;
   - trappole/insidie incontrate e come evitarle.
2. **Scrivi** queste indicazioni nel file giusto (senza duplicare ciò che c'è già):
   - **`MEMO.md`** → decisioni tecniche/architetturali, vincoli, fatti verificati, trucchi operativi;
   - **`ROADMAP.md`** → avanzamento (spunta gli item fatti) e prossimi passi;
   - **`CLAUDE.md`** (questo) → convenzioni di lavoro, preferenze dell'utente, come prendere le decisioni.
3. **Riassumi** in chat cosa hai scritto e dove.

> Obiettivo: ogni sessione lascia il progetto più "pronto" della precedente. Conciso e azionabile, niente muri di testo.

### Preferenze dell'utente (aggiornare qui col tempo)
- Vuole soluzioni **gratuite e senza carta di credito**; rifiuta i piani a pagamento se evitabili.
- Vuole **semplicità e leggibilità**, best practice dei framework; **niente over-engineering**.
- Si fida quando **verifico i fatti online** (specie cose che cambiano nel tempo) invece di andare a memoria.
- Fa **domande di pianificazione** prima di codare: meglio chiarire architettura/costi/rischi prima.
  Asseconda: sviscera alternative/costi/rischi **prima** di scrivere codice, senza fretta di codare.
- **Principiante sul backend** → spiegare i concetti in modo **elementare**, con analogie, e indicare il "perché" delle scelte.
- Segnalare sempre **refusi/errori** in italiano e inglese (anche fuori task) — vedi istruzioni globali.
- **"committa"** in questo progetto = **commit + push su `main`** (→ redeploy automatico su GitHub Pages). Non fermarsi al commit locale salvo richiesta esplicita.
- **"crea i pacchetti" / "pubblica gli installer"** = creare e pushare un **tag `v<versione>`** (deve corrispondere a `server/package.json`) → fa partire la CI Electron (`build-packages.yml`, matrix Win+Mac) e pubblica gli installer su **GitHub Releases**. Il solo push su `main` aggiorna **solo** Pages, NON ricostruisce l'app desktop.
- **Signals-first (preferenza esplicita 2026-07-01)**: l'utente preferisce i **signals** al posto di callback/stato imperativo. Per lo stato condiviso → **signal-store nel service** (fonte di verità unica, update ottimistici; i componenti leggono via `computed`), niente `load()` ripetuti. Timer/tempo restano necessariamente imperativi (es. scheduler), ma i dati che alimentano la UI sono signal. Esempio: `ScheduleService` (vedi `MEMO.md` §14).
- Apprezza la **verifica visiva** (screenshot headless della webapp) quando si toccano UI/stili. ⚠️ Attenzione alle trappole degli screenshot headless (vedi `MEMO.md` §7).
- **Versioning delegato a Claude**: bumpa `server/package.json` → `version` autonomamente nello stesso commit della modifica.
  - **patch** (0.1.x) → qualsiasi commit che produce una nuova build (bugfix, CI fix, refactor, script)
  - **minor** (0.x.0) → completamento di una Fase o feature rilevante
  - **major** (x.0.0) → solo su indicazione esplicita dell'utente
  - ⚠️ Bumpa **anche** `client/src/environments/environment.ts` **e** `environment.mock.ts` → campo `version` (mostrato nella UI sotto il titolo): va tenuto in sync con `server/package.json`.
  - ⚠️ Ogni volta che si triggera una nuova build (`yarn release` o tag), la versione deve essere già stata bumpata nel commit precedente.
  - Quando si crea un tag git, assicurarsi che corrisponda alla versione nel `package.json`.

## Come scrivere in MEMO/ROADMAP: separare «verificato» da «ipotesi»

Lezione pagata cara il 2026-07-29. Una voce di `MEMO.md` affermava come **acquisito**
(«`hardenedRuntime: false` è OBBLIGATORIO») quello che era solo una **congettura mai testata su un Mac**.
Era falsa, ha indirizzato male la sessione successiva e sono serviti due giri di build per smontarla.

- Marcare sempre lo stato di una conclusione: **✅ verificato** (con *come* è stato verificato e la data)
  oppure **⏳ ipotesi, non ancora confermata**. Mai scrivere una causa presunta come fosse dimostrata.
- Quando un'ipotesi viene smentita, **correggere la voce vecchia** (barrarla + rimando alla nuova), non
  solo aggiungerne una nuova: chi legge trova prima quella vecchia.
- Annotare le **ipotesi scartate e perché**: valgono quanto la soluzione, evitano di riproporle.

## Bug platform-specific (macOS) non riproducibili da Windows

Non tirare a indovinare a colpi di release: **far dimostrare il fatto alla CI**.

1. Aggiungere al workflow uno **step di ispezione** che stampa lo stato reale dell'artefatto
   (es. *"Inspect the Mac bundle"*: `codesign -dv` + `plutil -p` dell'Info.plist nel job macOS).
2. Iterare con **`gh workflow run build-packages.yml`** → costruisce e ispeziona in **~3 minuti**,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClemAnto/JingleMachine](https://github.com/ClemAnto/JingleMachine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
